---
title: IDEA 插件 Hello World
date: 2023-02-12 14:34:55
tags: 
- IDEA 插件
- Java
---

### 1. 工程结构
- HiClazz 是继承 AnAction 的实现类，用于附着到 IDEA 的窗体上，点击后打开对应页面
- MyDumbAwareAction、MyToolWindowFactory，配合使用，用于在 IDEA 最下面的窗体设置，与你看见的控制台输出信息位置一样。
- MySearchableConfigurable，可以用于 Settings 中配置窗体。
- TestUI 是基于 Swing 开发的窗体，验证在 AnAction 实现类中打开。
- plugin.xml 是整个 IDEA 咖啡的配置文件，你所有的窗体都会在这个配置文件里有所体现。

<!--more-->

```
PluginGuide
├── .gradle
└── src
    ├── main
    │   └── java
    │       ├── HiClazz.java
    │       ├── MyDumbAwareAction.java
    │       ├── MySearchableConfigurable.java
    │       ├── MyToolWindowFactory.java    
    │       └── TestUI.java    
    └── resources
        ├── icons  
        └── META-INF
            └── plugin.xml 

```

### 2. AnAction
```
public class HiClazz extends AnAction {

    @Override
    public void actionPerformed(AnActionEvent e) {
        Project project = e.getData(PlatformDataKeys.PROJECT);
        PsiFile psiFile = e.getData(CommonDataKeys.PSI_FILE);
        String classPath = psiFile.getVirtualFile().getPath();
        String title = "Hello World!";
        Messages.showMessageDialog(project, classPath, title, Messages.getInformationIcon());
    }

}

```

-   测试在 IDEA 中读取鼠标停留在类文件中的信息。我们可以把这个 AnAction 配置到各个 IDEA 菜单中。

### 3 plugin.xml


```
<extensions defaultExtensionNs="com.intellij">
    <!-- Add your extensions here -->
    <toolWindow canCloseContents="true" anchor="bottom"
                id="SmartIM"
                factoryClass="MyToolWindowFactory">
    </toolWindow>
    
    <!-- 在Setting中添加自定义配置模版 -->
    <projectConfigurable groupId="Other Settings" displayName="My Config" id="thief.id"
                         instance="MySearchableConfigurable"/>
</extensions>

<actions>
    <!-- Add your actions here -->
    <action id="HiId_FileMenu" class="HiClazz" text="HiName">
        <add-to-group group-id="FileMenu" anchor="first"/>
        <add-to-group group-id="MainMenu" anchor="first"/>
        <add-to-group group-id="EditMenu" anchor="first"/>
        <add-to-group group-id="ViewMenu" anchor="first"/>
        <add-to-group group-id="CodeMenu" anchor="first"/>
        <add-to-group group-id="AnalyzeMenu" anchor="first"/>
        <add-to-group group-id="RefactoringMenu" anchor="first"/>
        <add-to-group group-id="BuildMenu" anchor="first"/>
        <add-to-group group-id="RunMenu" anchor="first"/>
        <add-to-group group-id="ToolsMenu" anchor="first"/>
        <add-to-group group-id="WindowMenu" anchor="first"/>
        <add-to-group group-id="HelpMenu" anchor="first"/>
    </action>
    <action id="HiId_EditorPopupMenu" class="HiClazz" text="HiName">
        <add-to-group group-id="EditorPopupMenu" anchor="first"/>
    </action>
    <action id="HiId_MainToolBar" class="HiClazz" text="HiName">
        <add-to-group group-id="MainToolBar" anchor="first"/>
    </action>
</actions>

```
-   在 plugin.xml 的配置中，主要是把各个功能实现窗体配置到对应的菜单下，比如 Tools 下、toolWindow 里等。

### 4 测试结果
**启动运行**
![Gradle.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/518e5b65ac614b32b4e62956ce8169a9~tplv-k3u1fbpfcp-watermark.image?)
-   IDEA 插件开发运行会基于 Plugin 或者 Gradle 下配置的 `::runIde`

**运行效果**

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e35daca1a4004165ae99b38b42cfb831~tplv-k3u1fbpfcp-watermark.image?)
-   当鼠标点到类的上，在点 HiName 就可以看到对应的工程类信息了。

