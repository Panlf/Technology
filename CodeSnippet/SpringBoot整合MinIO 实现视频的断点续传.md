# SpringBoot 整合 MinIO 实现视频的分片上传/断点续传

## 数据库

|字段|注释|
|--|--|
|upload_id|分片上传的ID|
|file_identifier|文件唯一标识|
|file_name|文件名|
|bucket_name|所属桶名|
|object_key|文件的key|
|total_size|文件的大小|
|chunk_size|每个分片大小|
|chunk_num|分片数量|

## 后端实现
### 根据MD5获取是否存在相同文件
```
/**
 * 查询是否上传过，若存在，返回TaskInfoDTO
 * @param identifier 文件md5
 * @return
 */
@GetMapping("/{identifier}")
public GraceJSONResult taskInfo (@PathVariable("identifier") String identifier) {
    return GraceJSONResult.ok(sysUploadTaskService.getTaskInfo(identifier));
}
```

```
/**
 * 查询是否上传过，若存在，返回TaskInfoDTO
 * @param identifier
 * @return
 */
public TaskInfoDTO getTaskInfo(String identifier) {
    SysUploadTask task = getByIdentifier(identifier);
    if (task == null) {
        return null;
    }
    TaskInfoDTO result = new TaskInfoDTO().setFinished(true).setTaskRecord(TaskRecordDTO.convertFromEntity(task)).setPath(getPath(task.getBucketName(), task.getObjectKey()));

    boolean doesObjectExist = amazonS3.doesObjectExist(task.getBucketName(), task.getObjectKey());
    if (!doesObjectExist) {
        // 未上传完，返回已上传的分片
        ListPartsRequest listPartsRequest = new ListPartsRequest(task.getBucketName(), task.getObjectKey(), task.getUploadId());
        PartListing partListing = amazonS3.listParts(listPartsRequest);
        result.setFinished(false).getTaskRecord().setExitPartList(partListing.getParts());
    }
    return result;
} 
```

### 初始化一个上传任务
```
/**
 * 创建一个上传任务
 * @return
 */
@PostMapping
public GraceJSONResult initTask (@Valid @RequestBody InitTaskParam param) {
    return GraceJSONResult.ok(sysUploadTaskService.initTask(param));
}
```

```
/**
 * 初始化一个任务
 */
public TaskInfoDTO initTask(InitTaskParam param) {

    Date currentDate = new Date();
    String bucketName = minioProperties.getBucketName();
    String fileName = param.getFileName();
    String suffix = fileName.substring(fileName.lastIndexOf(".")+1, fileName.length());
    String key = StrUtil.format("{}/{}.{}", DateUtil.format(currentDate, "YYYY-MM-dd"), IdUtil.randomUUID(), suffix);
    String contentType = MediaTypeFactory.getMediaType(key).orElse(MediaType.APPLICATION_OCTET_STREAM).toString();
    ObjectMetadata objectMetadata = new ObjectMetadata();
    objectMetadata.setContentType(contentType);
    InitiateMultipartUploadResult initiateMultipartUploadResult = amazonS3
            .initiateMultipartUpload(new InitiateMultipartUploadRequest(bucketName, key).withObjectMetadata(objectMetadata));
    String uploadId = initiateMultipartUploadResult.getUploadId();

    SysUploadTask task = new SysUploadTask();
    int chunkNum = (int) Math.ceil(param.getTotalSize() * 1.0 / param.getChunkSize());
    task.setBucketName(minioProperties.getBucketName())
            .setChunkNum(chunkNum)
            .setChunkSize(param.getChunkSize())
            .setTotalSize(param.getTotalSize())
            .setFileIdentifier(param.getIdentifier())
            .setFileName(fileName)
            .setObjectKey(key)
            .setUploadId(uploadId);
    sysUploadTaskMapper.insert(task);
    return new TaskInfoDTO().setFinished(false).setTaskRecord(TaskRecordDTO.convertFromEntity(task)).setPath(getPath(bucketName, key));
}
```

### 获取每个分片的预签名上传地址
```
/**
 * 获取每个分片的预签名上传地址
 * @param identifier
 * @param partNumber
 * @return
 */
@GetMapping("/{identifier}/{partNumber}")
public GraceJSONResult preSignUploadUrl (@PathVariable("identifier") String identifier, @PathVariable("partNumber") Integer partNumber) {
    SysUploadTask task = sysUploadTaskService.getByIdentifier(identifier);
    if (task == null) {
        return GraceJSONResult.error("分片任务不存在");
    }
    Map<String, String> params = new HashMap<>();
    params.put("partNumber", partNumber.toString());
    params.put("uploadId", task.getUploadId());
    return GraceJSONResult.ok(sysUploadTaskService.genPreSignUploadUrl(task.getBucketName(), task.getObjectKey(), params));
}
```

```
/**
 * 生成预签名上传url
 * @param bucket 桶名
 * @param objectKey 对象的key
 * @param params 额外的参数
 * @return
 */
public String genPreSignUploadUrl(String bucket, String objectKey, Map<String, String> params) {
    Date currentDate = new Date();
    Date expireDate = DateUtil.offsetMillisecond(currentDate, PRE_SIGN_URL_EXPIRE.intValue());
    GeneratePresignedUrlRequest request = new GeneratePresignedUrlRequest(bucket, objectKey)
            .withExpiration(expireDate).withMethod(HttpMethod.PUT);
    if (params != null) {
        params.forEach((key, val) -> request.addRequestParameter(key, val));
    }
    URL preSignedUrl = amazonS3.generatePresignedUrl(request);
    return preSignedUrl.toString();
}
```

### 合并分片
```
/**
 * 合并分片
 * @param identifier
 * @return
 */
@PostMapping("/merge/{identifier}")
public GraceJSONResult merge (@PathVariable("identifier") String identifier) {
    sysUploadTaskService.merge(identifier);
    return GraceJSONResult.ok();
}
```

```
/**
 * 合并分片
 * @param identifier
 */
public void merge(String identifier) {
    SysUploadTask task = getByIdentifier(identifier);
    if (task == null) {
        throw new RuntimeException("分片任务不存");
    }

    ListPartsRequest listPartsRequest = new ListPartsRequest(task.getBucketName(), task.getObjectKey(), task.getUploadId());
    PartListing partListing = amazonS3.listParts(listPartsRequest);
    List<PartSummary> parts = partListing.getParts();
    if (!task.getChunkNum().equals(parts.size())) {
        // 已上传分块数量与记录中的数量不对应，不能合并分块
        throw new RuntimeException("分片缺失，请重新上传");
    }
    CompleteMultipartUploadRequest completeMultipartUploadRequest = new CompleteMultipartUploadRequest()
            .withUploadId(task.getUploadId())
            .withKey(task.getObjectKey())
            .withBucketName(task.getBucketName())
            .withPartETags(parts.stream().map(partSummary -> new PartETag(partSummary.getPartNumber(), partSummary.getETag())).collect(Collectors.toList()));
    CompleteMultipartUploadResult result = amazonS3.completeMultipartUpload(completeMultipartUploadRequest);
}
```

## 分片文件清理问题
视频上传一半不上传了，怎么清理碎片分片。

可以考虑在sys_upload_task表中新加一个status字段，表示是否合并分片，默认为false，merge请求结束后变更为true，通过一个定时任务定期清理为status为false的记录。另外MinIO自身对于临时上传的分片，会实施定时清理。