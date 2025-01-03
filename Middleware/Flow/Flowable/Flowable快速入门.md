# Flowable快速入门

- RepositoryService很可能是使用Flowable引擎要用的第一个服务。这个服务提供了管理与控制部署(deployments)与流程定义(process definitions)的操作。管理静态信息
- RuntimeService用于启动流程定义的新流程实例
- IdentityService很简单。它用于管理（创建，更新，删除，查询……）组与用户
- FormService是可选服务。也就是说Flowable没有它也能很好地运行，而不必牺牲任何功能
- HistoryService暴露Flowable引擎收集的所有历史数据。要提供查询历史数据的能力
- ManagementService通常在用Flowable编写用户应用时不需要使用。它可以读取数据库表与表原始数据的信息，也提供了对作业(job)的查询与管理操作
- DynamicBpmnService可用于修改流程定义中的部分内容，而不需要重新部署它。例如可以修改流程定义中一个用户任务的办理人设置，或者修改一个服务任务中的类名

快速入门代码
```java
import lombok.extern.slf4j.Slf4j;
import org.flowable.engine.HistoryService;
import org.flowable.engine.RepositoryService;
import org.flowable.engine.RuntimeService;
import org.flowable.engine.history.HistoricProcessInstance;
import org.flowable.engine.repository.Deployment;
import org.flowable.engine.repository.ProcessDefinition;
import org.flowable.engine.runtime.Execution;
import org.flowable.engine.runtime.ProcessInstance;
import org.flowable.idm.api.Group;
import org.flowable.idm.api.User;
import org.flowable.task.api.Task;
import org.flowable.task.api.history.HistoricTaskInstance;
import org.springframework.beans.factory.annotation.Autowired;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.zip.ZipInputStream;


@Slf4j
public class TestFlowable {

    @Autowired
    private RepositoryService repositoryService;

    @Autowired
    private RuntimeService runtimeService;

    @Autowired
    private HistoryService historyService;

    @Autowired
    private org.flowable.engine.TaskService taskService;

    @Autowired
    private org.flowable.engine.IdentityService identityService;

    public void createDeploymentZip() {

        /*
         * @Date: 2021/10/17 23:38
         * Step 1: 部署xml（压缩到zip形式，直接xml需要配置相对路径，麻烦，暂不用）
         */
        try {
            File zipTemp = new File("f:/leave_approval.bpmn20.zip");
            ZipInputStream zipInputStream = new ZipInputStream(new FileInputStream(zipTemp));
            Deployment deployment = repositoryService
                    .createDeployment()
                    .addZipInputStream(zipInputStream)
                    .deploy();
            log.info("部署成功:{}", deployment.getId());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }

        /*
         * @Date: 2021/10/17 23:40
         * Step 2: 查询部署的流程定义
         */
        List<ProcessDefinition> list = repositoryService.createProcessDefinitionQuery().processDefinitionKey("leave_approval").list();
        List<ProcessDefinition> pages = repositoryService.createProcessDefinitionQuery().processDefinitionKey("leave_approval").listPage(1, 30);

        /*
         * @Date: 2021/10/17 23:40
         * Step 3: 启动流程，创建实例
         */
        String processDefinitionKey = "leave_approval";//流程定义的key,对应请假的流程图
        String businessKey = "schoolleave";//业务代码，根据自己的业务用
        Map<String, Object> variablesDefinition = new HashMap<>();//流程变量，可以自定义扩充
        ProcessInstance processInstance = runtimeService.startProcessInstanceByKey(processDefinitionKey, businessKey, variablesDefinition);
        log.info("启动成功:{}", processInstance.getId());

        /*
         * @Date: 2021/10/17 23:40
         * Step 4: 查询指定流程所有启动的实例列表
         * 列表，或 分页 删除
         */
        List<Execution> executions = runtimeService.createExecutionQuery().processDefinitionKey("leave_approval").list();
        List<Execution> executionPages = runtimeService.createExecutionQuery().processDefinitionKey("leave_approval").listPage(1, 30);
//        runtimeService.deleteProcessInstance(processInstanceId, deleteReason); //删除实例

        /*
         * @Date: 2021/10/17 23:40
         * Step 5: 学生查询可以操作的任务,并完成任务
         */
        String candidateGroup = "stu_group"; //候选组 xml文件里面的 flowable:candidateGroups="stu_group"
        List<Task> taskList = taskService.createTaskQuery().taskCandidateGroup(candidateGroup).orderByTaskCreateTime().desc().list();
        for (Task task : taskList) {
            // 申领任务
            taskService.claim(task.getId(), "my");
            // 完成
            taskService.complete(task.getId());
        }

        /*
         * @Date: 2021/10/17 23:40
         * Step 6: 老师查询可以操作的任务,并完成任务
         */
        String candidateGroupTe = "te_group"; //候选组 xml文件里面的 flowable:candidateGroups="te_group"
        List<Task> taskListTe = taskService.createTaskQuery().taskCandidateGroup(candidateGroupTe).orderByTaskCreateTime().desc().list();
        for (Task task : taskListTe) {
            // 申领任务
            taskService.claim(task.getId(), "myte");
            // 完成
            Map<String, Object> variables = new HashMap<>();
            variables.put("command","agree"); //携带变量，用于网关流程的条件判定，这里的条件是同意
            taskService.complete(task.getId(), variables);
        }

        /*
         * @Date: 2021/10/18 0:17
         * Step 7: 历史查询，因为一旦流程执行完毕，活动的数据都会被清空，上面查询的接口都查不到数据，但是提供历史查询接口
         */
        // 历史流程实例
        List<HistoricProcessInstance> historicProcessList = historyService.createHistoricProcessInstanceQuery().processDefinitionKey("leave_approval").list();
        // 历史任务
        List<HistoricTaskInstance> historicTaskList = historyService.createHistoricTaskInstanceQuery().processDefinitionKey("leave_approval").list();
        // 实例历史变量 , 任务历史变量
        // historyService.createHistoricVariableInstanceQuery().processInstanceId(processInstanceId);
        // historyService.createHistoricVariableInstanceQuery().taskId(taskId);

        // *****************************************************分隔符********************************************************************
        // *****************************************************分隔符********************************************************************
        // 可能还需要的API
        // 移动任务，人为跳转任务
        // runtimeService.createChangeActivityStateBuilder().processInstanceId(processInstanceId)
        //       .moveActivityIdTo(currentActivityTaskId, newActivityTaskId).changeState();

        // 如果在数据库配置了分组和用户，还会用到
        List<User> users = identityService.createUserQuery().list();    //用户查询，用户id对应xml 里面配置的用户
        List<Group> groups = identityService.createGroupQuery().list(); //分组查询，分组id对应xml 里面配置的分组 如 stu_group，te_group 在表里是id的值

        // 另外，每个查询后面都可以拼条件，内置恁多查询，包括模糊查询，大小比较都有
    }
}
```