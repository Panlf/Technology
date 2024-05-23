# PDF添加水印

## 使用 Apache PDFBox 库
```
<dependency>
    <groupId>org.apache.pdfbox</groupId>
    <artifactId>pdfbox</artifactId>
    <version>2.0.24</version>
</dependency>
```

在添加水印之前，需要读取原始 PDF 文件：
```
PDDocument document = PDDocument.load(new File("original.pdf"));

```

然后，遍历 PDF 中的所有页面，并使用 PDPageContentStream 添加水印：
```
// 遍历 PDF 中的所有页面
for (int i = 0; i < document.getNumberOfPages(); i++) {
    PDPage page = document.getPage(i);
    PDPageContentStream contentStream = new PDPageContentStream(document, page, PDPageContentStream.AppendMode.APPEND, true, true);

    // 设置字体和字号
    contentStream.setFont(PDType1Font.HELVETICA_BOLD, 36);

    // 设置透明度
    contentStream.setNonStrokingColor(200, 200, 200);

    // 添加文本水印
    contentStream.beginText();
    contentStream.newLineAtOffset(100, 100); // 设置水印位置
    contentStream.showText("Watermark"); // 设置水印内容
    contentStream.endText();

    contentStream.close();
}
```

最后，需要保存修改后的 PDF 文件：
```
document.save(new File("output.pdf"));
document.close();
```

完整代码
```
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.PDPageContentStream;
import org.apache.pdfbox.pdmodel.font.PDType1Font;

import java.io.File;
import java.io.IOException;

public class PdfBoxWatermark {
    public static void main(String[] args) throws IOException {
        // 读取原始 PDF 文件
        PDDocument document = PDDocument.load(new File("original.pdf"));

        // 遍历 PDF 中的所有页面
        for (int i = 0; i < document.getNumberOfPages(); i++) {
            PDPage page = document.getPage(i);
            PDPageContentStream contentStream = new PDPageContentStream(document, page, PDPageContentStream.AppendMode.APPEND, true, true);

            // 设置字体和字号
            contentStream.setFont(PDType1Font.HELVETICA_BOLD, 36);

            // 设置透明度
            contentStream.setNonStrokingColor(200, 200, 200);

            // 添加文本水印
            contentStream.beginText();
            contentStream.newLineAtOffset(100, 100); // 设置水印位置
            contentStream.showText("Watermark"); // 设置水印内容
            contentStream.endText();

            contentStream.close();
        }

        // 保存修改后的 PDF 文件
        document.save(new File("output.pdf"));
        document.close();
    }
}
```
## 使用 iText 库
在 pom.xml 文件中添加 iText 的依赖：
```
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itextpdf</artifactId>
    <version>5.5.13</version>
</dependency>
```

在添加水印之前，需要读取原始 PDF 文件：
```
PdfReader reader = new PdfReader("original.pdf");
PdfStamper stamper = new PdfStamper(reader, new FileOutputStream("output.pdf"));
```

然后，遍历 PDF 中的所有页面，并使用 PdfContentByte 添加水印：
```
// 获取 PDF 中的页数
int pageCount = reader.getNumberOfPages();

// 添加水印
for (int i = 1; i <= pageCount; i++) {
    PdfContentByte contentByte = stamper.getUnderContent(i); // 或者 getOverContent()
    contentByte.beginText();
    contentByte.setFontAndSize(BaseFont.createFont(), 36f);
    contentByte.setColorFill(BaseColor.LIGHT_GRAY);
    contentByte.showTextAligned(Element.ALIGN_CENTER, "Watermark", 300, 400, 45);
    contentByte.endText();
}
```

最后，需要保存修改后的 PDF 文件并关闭文件流：
```
stamper.close();
reader.close();
```

完整代码
```
import com.itextpdf.text.*;
import com.itextpdf.text.pdf.*;

import java.io.FileOutputStream;
import java.io.IOException;

public class ItextWatermark {
    public static void main(String[] args) throws IOException, DocumentException {
        // 读取原始 PDF 文件
        PdfReader reader = new PdfReader("original.pdf");
        PdfStamper stamper = new PdfStamper(reader, new FileOutputStream("output.pdf"));

        // 获取 PDF 中的页数
        int pageCount = reader.getNumberOfPages();

        // 添加水印
        for (int i = 1; i <= pageCount; i++) {
            PdfContentByte contentByte = stamper.getUnderContent(i); // 或者 getOverContent()
            contentByte.beginText();
            contentByte.setFontAndSize(BaseFont.createFont(), 36f);
            contentByte.setColorFill(BaseColor.LIGHT_GRAY);
            contentByte.showTextAligned(Element.ALIGN_CENTER, "Watermark", 300, 400, 45);
            contentByte.endText();
        }

        // 保存修改后的 PDF 文件并关闭文件流
        stamper.close();
        reader.close();
    }
}
```

## Free Spire.PDF for Java

```
<dependency>
    <groupId>e-iceblue</groupId>
    <artifactId>free-spire-pdf-for-java</artifactId>
    <version>1.9.6</version>
</dependency>
```

在添加水印之前，需要读取原始 PDF 文件：
```
PdfDocument pdf = new PdfDocument();
pdf.loadFromFile("original.pdf");
```

然后，遍历 PDF 中的所有页面，并使用 PdfPageBase 添加水印：
```
// 遍历 PDF 中的所有页面
for (int i = 0; i < pdf.getPages().getCount(); i++) {
    PdfPageBase page = pdf.getPages().get(i);

    // 添加文本水印
    PdfWatermark watermark = new PdfWatermark("Watermark");
    watermark.setFont(new PdfFont(PdfFontFamily.Helvetica, 36));
    watermark.setOpacity(0.5f);
    page.getWatermarks().add(watermark);
}
```
最后，需要保存修改后的 PDF 文件：
```

pdf.saveToFile("output.pdf");
pdf.close();
```

添加图片水印与添加文本水印类似，只需要将 PdfWatermark 的参数修改为图片路径即可。
```
// 添加图片水印
PdfWatermark watermark = new PdfWatermark("watermark.png");
watermark.setOpacity(0.5f);
page.getWatermarks().add(watermark);
```

完整代码
```
import com.spire.pdf.*;

public class FreeSpirePdfWatermark {
    public static void main(String[] args) {
        // 读取原始 PDF 文件
        PdfDocument pdf = new PdfDocument();
        pdf.loadFromFile("original.pdf");

        // 遍历 PDF 中的所有页面
        for (int i = 0; i < pdf.getPages().getCount(); i++) {
            PdfPageBase page = pdf.getPages().get(i);

            // 添加文本水印
            PdfWatermark watermark = new PdfWatermark("Watermark");
            watermark.setFont(new PdfFont(PdfFontFamily.Helvetica, 36));
            watermark.setOpacity(0.5f);
            page.getWatermarks().add(watermark);

            // 添加图片水印
            // PdfWatermark watermark = new PdfWatermark("watermark.png");
            // watermark.setOpacity(0.5f);
            // page.getWatermarks().add(watermark);
        }

        // 保存修改后的 PDF 文件
        pdf.saveToFile("output.pdf");
        pdf.close();
    }
}
```

## Aspose.PDF for Java

```
<dependency>
    <groupId>com.aspose</groupId>
    <artifactId>aspose-pdf</artifactId>
    <version>21.4</version>
</dependency>
```

添加文本水印
```
@PostMapping("/addTextWatermark")
public ResponseEntity<byte[]> addTextWatermark(@RequestParam("file") MultipartFile file) throws IOException {
    // 加载 PDF 文件
    Document pdfDocument = new Document(file.getInputStream());
    TextStamp textStamp = new TextStamp("Watermark");
    textStamp.setWordWrap(true);
    textStamp.setVerticalAlignment(VerticalAlignment.Center);
    textStamp.setHorizontalAlignment(HorizontalAlignment.Center);
    pdfDocument.getPages().get_Item(1).addStamp(textStamp);

    // 保存 PDF 文件
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    pdfDocument.save(outputStream);
    return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"watermarked.pdf\"")
            .contentType(MediaType.APPLICATION_PDF)
            .body(outputStream.toByteArray());
}
```

添加图片水印
```
@PostMapping("/addImageWatermark")
public ResponseEntity<byte[]> addImageWatermark(@RequestParam("file") MultipartFile file) throws IOException {
    // 加载 PDF 文件
    Document pdfDocument = new Document(file.getInputStream());
    ImageStamp imageStamp = new ImageStamp("watermark.png");
    imageStamp.setWidth(100);
    imageStamp.setHeight(100);
    imageStamp.setVerticalAlignment(VerticalAlignment.Center);
    imageStamp.setHorizontalAlignment(HorizontalAlignment.Center);
    pdfDocument.getPages().get_Item(1).addStamp(imageStamp);

    // 保存 PDF 文件
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    pdfDocument.save(outputStream);
    return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"watermarked.pdf\"")
            .contentType(MediaType.APPLICATION_PDF)
            .body(outputStream.toByteArray());
}
```

完整代码
```
import com.aspose.pdf.*;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.ByteArrayOutputStream;
import java.io.IOException;

@RestController
@RequestMapping("/api/pdf")
public class PdfController {
    @PostMapping("/addTextWatermark")
    public ResponseEntity<byte[]> addTextWatermark(@RequestParam("file") MultipartFile file) throws IOException {
        // 加载 PDF 文件
        Document pdfDocument = new Document(file.getInputStream());
        TextStamp textStamp = new TextStamp("Watermark");
        textStamp.setWordWrap(true);
        textStamp.setVerticalAlignment(VerticalAlignment.Center);
        textStamp.setHorizontalAlignment(HorizontalAlignment.Center);
        pdfDocument.getPages().get_Item(1).addStamp(textStamp);

        // 保存 PDF 文件
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        pdfDocument.save(outputStream);
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"watermarked.pdf\"")
                .contentType(MediaType.APPLICATION_PDF)
                .body(outputStream.toByteArray());
    }

    @PostMapping("/addImageWatermark")
    public ResponseEntity<byte[]> addImageWatermark(@RequestParam("file") MultipartFile file) throws IOException {
        // 加载 PDF 文件
        Document pdfDocument = new Document(file.getInputStream());
        ImageStamp imageStamp = new ImageStamp("watermark.png");
        imageStamp.setWidth(100);
        imageStamp.setHeight(100);
        imageStamp.setVerticalAlignment(VerticalAlignment.Center);
        imageStamp.setHorizontalAlignment(HorizontalAlignment.Center);
        pdfDocument.getPages().get_Item(1).addStamp(imageStamp);

        // 保存 PDF 文件
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        pdfDocument.save(outputStream);
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"watermarked.pdf\"")
                .contentType(MediaType.APPLICATION_PDF)
                .body(outputStream.toByteArray());
    }
}
```