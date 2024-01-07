---
title: Kernel Memory 入门系列：自定义处理流程
date: 2023-12-27 20:00:00
categories:
  - Kernel Memory
tags:
  - StepName
  - Pipeline
  - InvokeAsync
description: Kernel Memory 入门系列：自定义处理流程
cover: https://s2.loli.net/2024/01/07/TS9UXHiKwAjZ4s3.png
---
![image.png](https://s2.loli.net/2024/01/07/TS9UXHiKwAjZ4s3.png)
## Kernel Memory 入门系列：自定义处理流程
在整个文档预处理的流程中，涉及到很多的处理步骤，例如：文本提取，文本分片，向量化和存储。这些步骤是Kernel Memory中的默认提供的处理方法，如果有一些其他的需求，也可以进行过程的自定义。

## 自定义Handler
在Kernel Memory中，可以通过自定义Handler的方式来实现自定义的处理流程。自定义Handler需要实现IPipelineStepHandler接口，该接口定义如下：
```undefined
public interface IPipelineStepHandler
{
    string StepName { get; }
 
    Task<(bool success, DataPipeline updatedPipeline)> InvokeAsync(DataPipeline pipeline, CancellationToken cancellationToken = default);
}
```
其中，StepName是自定义Handler的名称，用于在Pipeline中指定该步骤，InvokeAsync方法是自定义Handler的执行方法。在InvokeAsync方法中，可以对DataPipeline中的数据进行修改，从而实现自定义的处理。

主要的实现逻辑在InvokeAsync方法中，其中DataPipeline是主要的数据结构，它包含了整个文档处理的流程中的所有数据。

如果想要得到当前处理流程中的文件，可以通过DataPipeline.Files属性获取。
```undefined
public async Task<(bool success, DataPipeline updatedPipeline)> InvokeAsync(DataPipeline pipeline, CancellationToken cancellationToken = default)
{
    foreach (DataPipeline.FileDetails file in pipeline.Files)
    {
        Console.WriteLine(file.Name);
    }
 
    return (true, pipeline);
}
```
实现的过程中建议为Handler注入IPipelineOrchestrator, 通过IPipelineOrchestrator可以获取到当前的Memory的大部分基础组件和文件管理的方法。

例如，如果想获取文件的内容，就可以IPipelineOrchestrator.ReadTextFileAsync方法：
```undefined
IPipelineOrchestrator orchestrator;
var fileContent = await orchestrator.ReadTextFileAsync(pipeline, file.Name, cancellationToken);
```
如果想要存储生成的文件内容，就可以使用IPipelineOrchestrator.WriteTextFileAsync方法：
```undefined
IPipelineOrchestrator orchestrator;
await orchestrator.WriteTextFileAsync(pipeline, file.Name, fileContent, cancellationToken);
```
除了文本内容，还可以通过IPipelineOrchestrator.ReadFileAsync和IPipelineOrchestrator.WriteFileAsync方法来读取和存储二进制文件。

除此之外，还可以通过IPipelineOrchestrator 获取 TextGenerator 、EmbeddingGenerators、MemoryDbs等基础组件，搭配使用实现更多丰富的流程。

例如使用TextGenerator文本生成服务，可以构建自己的提示词方法，为当前的文档生成摘要、提炼关键词等。

生成的文本，首先通过IPipelineOrchestrator.WriteFileAsync进行内容的存储， 然后将文件信息存放到file.GeneratedFiles中，这样就可以在后续的处理流程中使用了。
```undefined
var generatedFile = ...;
await orchestrator.WriteFileAsync(pipeline, generatedFile.Name, generatedFile.Content, cancellationToken);
file.GeneratedFiles.Add(new DataPipeline.GeneratedFileDetails
                {
                    Id = Guid.NewGuid().ToString("N"),
                    ParentId = file.Id,
                    Name = generatedFile.Name,
                    Size = generatedFile.Length,
                    MimeType = generatedFile.Type,
                    ArtifactType = DataPipeline.ArtifactTypes.SyntheticData,
                    Tags = pipeline.Tags,
                });
```
另外，其中的File本身存在一组方法，可以用来判断该文件是否已经被当前流程处理过了，以避免重复处理：
```undefined
 file.MarkProcessedBy(this); // 标记当前文件已经被当前Handler处理过了
 file.AlreadyProcessedBy(this); // 判断当前文件是否已经被当前Handler处理过了
```
## 注册Handler
完成Handler逻辑的编写后，就可以将Handler注册到Memory中进行使用了。

在构建Memory后，通过AddHandler方法即可完成注册：
```undefined
var memory = new KernelMemoryBuilder()
    // ...
    .Build<MemoryServerless>();
 
memory.AddHandler(new MyHandler(memory.Orchestrator));
```
另外也可以在 MemoryBuilder 阶段，针对Orchestrator进行Handler的注册：
```undefined
var memoryBuilder = new KernelMemoryBuilder();
var orchestrator = memoryBuilder.GetOrchestrator();
 
var myHandler = new MyHandler(orchestrator);
await orchestrator.AddHandlerAsync(myHandler);
```
## 自定义处理流程
注册完成Handler后，就可以在自定义的Pipeline中使用了。

一种方式是在memory.ImportDocumentAsync的时候，指定 Steps:
```undefined
await memory.ImportDocumentAsync("sample-Wikipedia-Moon.txt", steps: new[] { "my_step" });
```
另一种是围绕着Orchestrator进行Pipeline的构建：
```undefined
var pipeline = orchestrator
    .PrepareNewDocumentUpload(index: "tests", documentId: "inProcessTest", new TagCollection { { "testName", "example3" } })
    .AddUploadFile("file1", "file1-Wikipedia-Carbon.txt", "file1-Wikipedia-Carbon.txt")
    .AddUploadFile("file2", "file2-Wikipedia-Moon.txt", "file2-Wikipedia-Moon.txt")
    .Then("extract")
    .Then("partition")
    .Then("summarize")
    .Then("gen_embeddings")
    .Then("save_records")
    .Build();
 
await orchestrator.RunPipelineAsync(pipeline, cancellationToken);
```
以上就完成了自定义流程的实现。

**参考**

InProcessMemoryWithCustomHandler:

**https://github.com/microsoft/kernel-memory/tree/main/examples/201-dotnet-InProcessMemoryWithCustomHandler**

ServerlessCustomPipeline:
**https://github.com/microsoft/kernel-memory/tree/main/examples/004-dotnet-ServerlessCustomPipeline**
