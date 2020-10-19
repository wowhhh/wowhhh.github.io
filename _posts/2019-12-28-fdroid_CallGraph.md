---
layout: post
title:  "FlowDroid-生成函数调用图"
tags:   StaticAnalysis CallGraph
date:   2019-12-28 10:10:35 +0800
categories: [FlowDroid]
---

## 本文目的

1：使用Flowdroid，对APK进行分析，提取出函数调用图

2：对函数调用图进行分析，提取出APK的所有Call Chain

3：对Call Chain进行可视化

除以上外，在细节部分也会涉及Soot的初始化等内容。

## Call Graph

#### CG是什么？

CG为call graph，表示函数调用图，表示整个程序中方法（函数）之间调用关系的图。例如onCreate()方法中调用了foo()，那么在CG图中就存在一个从onCreate()到foo()的边。

#### 如何提取出CG

- **Call Graoh Algorithm**

  CHA、GEOM、RTA、VTA、SPARK

- **SetupApplication**

  值得注意：第二个代码片段中许多语句是用于配置Soot的，如果想简单的实现生成Call Graph的操作可以使用：

  ```java
  SetupApplication application = new SetupApplication(androidJarPath,
                                                              apkPath);
  soot.G.reset();
  
  application.setCallbackFile(Config.get().androidCallbacksFile.toString());//设置预先准备的Android Callback文件
  ```

  现在再看下述配置，可以根据自身程序进行调整：

  ```java
  SetupApplication application = new SetupApplication(androidJarPath,
                                                              apkPath);
  soot.G.reset();
  SootConfigForAndroid sootConf = new SootConfigForAndroid() {
         @Override
         public void setSootOptions(Options options, InfoflowConfiguration config) {
             super.setSootOptions(options, config);
             config.setCallgraphAlgorithm(cgAlgo);//指定提取Call Graph的算法
             Options.v().set_allow_phantom_refs(true);//表示是否加载未被解析的类
             Options.v().set_whole_program(true);//以全局应用的模式运行
             Options.v().set_prepend_classpath(true);//使用配置好的classpath路径
             Options.v().set_process_multiple_dex(true);//是否处理多个dex文件
             Options.v().set_validate(true);
           Options.v().set_force_android_jar(Config.get().versionSdkFile.toString());//在该路径下寻找android.jar文件
                 Options.v().set_soot_classpath(Config.get().versionSdkFile.toString());
                 Options.v().set_process_dir(Collections.singletonList(Config.get().apkFile.toString()));//apk文件地址
             Options.v().set_src_prec(inputFormat);//表示反编译后文件的生成文件类型
             Options.v().set_output_format(Options.output_format_dex);//设置输出的格式
              }
          };
  application.setSootConfig(sootConf);//将soot设置的配置，应用到SetupApplication
       application.setCallbackFile(Config.get().androidCallbacksFile.toString());//设置预先准备的Android Callback文件
  Scene.v().loadNecessaryClasses();//加载类，这里加载的应该是java jar包中的rt.jar
  ```

- **Construct Call Graph**

  ```java
  application.constructCallgraph();//构建
  CallGraph callGraph = Scene.v().getCallGraph();//获取
  ```

- **生成函数调用图的极简形式**

  ```java
  /**生成函数调用图*/
  public void generateCallGraph(String androidJarPath ,String apkPath )
  {
      SetupApplication setupApplication = new SetupApplication(androidJarPath,apkPath);// 提供android.jar和apk的存放路径
      soot.G.reset();//reset
    setupApplication.setCallbackFile(Main.class.getResource("AndroidCallbacks.txt").getFile()); //
      setupApplication.constructCallgraph(); //开始构建调用图
      CallGraph callGraph = Scene.v().getCallGraph(); //获取，也可以把callGraph保存到文件里面
      System.out.println(callGraph);
  }
  ```

  保存好之后就是下面这种：

  ![](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/markdown-img-paste-20191228195703974.png)

## Call Chain

如上图，因为Soot的解析是多线程的，所以得到的Call Graph的边就很乱，没有顺序，那如何把这些调用关系给串联起来，得到函数之间的调用顺序。

首先明确在Call Graph中我们有什么信息？答：edge，并且每条边都有source和target。那就需要根据这个信息，把需要去遍历寻找Call Chain的方法寻找出来：

```java
for (Edge edge : cg) {
    String tgtSig = edge.getTgt().method().getSignature();//获取边的target
    apkMethods.add(tgtSig);
}
```

有了方法之后，就需要回溯的去寻找谁调用了这个方法：

参考链接：[CFG of android app](https://github.com/secure-software-engineering/soot-infoflow-android/issues/155#issuecomment-344506022)

```java
private static void visitMethod(CallGraph cg, SootMethod method) {
  visited.put(method.getSignature(), true);//标记是否已经遍历过
  Iterator<MethodOrMethodContext> ptargets = new Sources(cg.edgesInto(method));
  if (ptargets != null) {
    while (ptargets.hasNext()) {
        SootMethod parent = (SootMethod) ptargets.next();
        if (!visited.containsKey(parent.getSignature())) visitMethod(cg, parent);
    }
} 
```

## 如何可视化？

参考文章：[使用FlowDroid生成Android应用程序的函数调用图](https://blog.csdn.net/liu3237/article/details/48827523)

整个可视化的过程就是在上面去遍历寻找Call Chain的时候，把每个方法做成结点，之间的关系做成边，然后加入到gexf格式的图中。

- Step1:

  ```java
  public static void main(String[] args)
     {
         SetupApplication app = new SetupApplication(jarPath, apk);
         soot.G.reset();
         //传入AndroidCallbacks.txt文件
         app.setCallbackFile(Generator.class.getResource("AndroidCallbacks.txt").getFile());
         app.constructCallgraph();
  
         //SootMethod获取函数调用图
         SootMethod entryPoint = app.getDummyMainMethod();// dummy main method
         CallGraph cg = Scene.v().getCallGraph();
         System.out.println(cg);
         //可视化
        visit(cg,entryPoint);
         //导出函数调用图
        cge.exportMIG("flowdroidCFG", "D:/");
     }
  ```

  ```java
     //可视化函数调用图的函数
      private static void visit(CallGraph cg,SootMethod m){
          //在soot中，函数的signature就是由该函数的类名，函数名，参数类型，以及返回值类型组成的字符串
          String identifier = m.getSignature();
          //记录是否已经处理过该点
          visited.put(identifier, true);
          //以函数的signature为label在图中添加该节点
          cge.createNode(identifier);
          //获取调用该函数的函数 source
          Iterator<MethodOrMethodContext> ptargets = new Targets(cg.edgesInto(m));
          if(ptargets != null){
              while(ptargets.hasNext())
              {
                  SootMethod p = (SootMethod) ptargets.next();
                  if(p == null){
                      System.out.println("p is null");
                  }
                  if(!visited.containsKey(p.getSignature())){
                      visit(cg,p);
                  }
              }
          }
          //获取该函数调用的函数 target
          Iterator<MethodOrMethodContext> ctargets = new Targets(cg.edgesOutOf(m));
          if(ctargets != null){
              while(ctargets.hasNext())
              {
                  SootMethod c = (SootMethod) ctargets.next();
                  if(c == null){
                      System.out.println("c is null");
                  }
                  //将被调用的函数加入图中
                  cge.createNode(c.getSignature());
                  //添加一条指向该被调函数的边
                  cge.linkNodeByID(identifier, c.getSignature());
                  if(!visited.containsKey(c.getSignature())){
                      //递归
                      visit(cg,c);
                  }
              }
          }
      }
  ```

  
- Step2:

  emmm，这个画图的懒得分析了（其实是不会），逃了~

  ```java
  import it.uniroma1.dis.wsngroup.gexf4j.core.EdgeType;
  import it.uniroma1.dis.wsngroup.gexf4j.core.Gexf;
  import it.uniroma1.dis.wsngroup.gexf4j.core.Graph;
  import it.uniroma1.dis.wsngroup.gexf4j.core.Mode;
  import it.uniroma1.dis.wsngroup.gexf4j.core.Node;
  import it.uniroma1.dis.wsngroup.gexf4j.core.data.Attribute;
  import it.uniroma1.dis.wsngroup.gexf4j.core.data.AttributeClass;
  import it.uniroma1.dis.wsngroup.gexf4j.core.data.AttributeList;
  import it.uniroma1.dis.wsngroup.gexf4j.core.data.AttributeType;
  import it.uniroma1.dis.wsngroup.gexf4j.core.impl.GexfImpl;
  import it.uniroma1.dis.wsngroup.gexf4j.core.impl.StaxGraphWriter;
  import it.uniroma1.dis.wsngroup.gexf4j.core.impl.data.AttributeListImpl;
  
  import java.io.File;
  import java.io.FileWriter;
  import java.io.IOException;
  import java.io.Writer;
  import java.util.List;
  public class CGExporter {
      private Gexf gexf;
      private Graph graph;
      private Attribute codeArray;
      private AttributeList attrList;
  
      public CGExporter() {
          this.gexf = new GexfImpl();
          this.graph = this.gexf.getGraph();
          this.gexf.getMetadata().setCreator("wwww").setDescription("App method invoke graph");
          this.gexf.setVisualization(true);
          this.graph.setDefaultEdgeType(EdgeType.DIRECTED).setMode(Mode.STATIC);
          this.attrList = new AttributeListImpl(AttributeClass.NODE);
          this.graph.getAttributeLists().add(attrList);
          //可以给每个节点设置一些属性，这里设置的属性名是 codeArray，实际上后面没用到
          this.codeArray = this.attrList.createAttribute("0", AttributeType.STRING,"codeArray");
      }
  
      public void exportMIG(String graphName, String storeDir) {
          String outPath = storeDir + "/" + graphName + ".gexf";
          StaxGraphWriter graphWriter = new StaxGraphWriter();
          File f = new File(outPath);
          Writer out;
          try {
              out = new FileWriter(f, false);
              graphWriter.writeToStream(this.gexf, out, "UTF-8");
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  
      public Node getNodeByID(String Id) {
          List<Node> nodes = this.graph.getNodes();
          Node nodeFinded = null;
          for (Node node : nodes) {
              String nodeID = node.getId();
              if (nodeID.equals(Id)) {
                  nodeFinded = node;
                  break;
              }
          }
          return nodeFinded;
      }
  
      public void linkNodeByID(String sourceID, String targetID) {
          Node sourceNode = this.getNodeByID(sourceID);
          Node targetNode = this.getNodeByID(targetID);
          if (sourceNode.equals(targetNode)) {
              return;
          }
          if (!sourceNode.hasEdgeTo(targetID)) {
              String edgeID = sourceID + "-->" + targetID;
              sourceNode.connectTo(edgeID, "", EdgeType.DIRECTED, targetNode);
          }
      }
  
      public void createNode(String m) {
          String id = m;
          String codes = "";
          if (getNodeByID(id) != null) {
              return;
          }
          Node node = this.graph.createNode(id);
          node.setLabel(id).getAttributeValues().addValue(this.codeArray, codes);
          node.setSize(20);
      }
  
  }
  ```

  

