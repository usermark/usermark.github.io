---
title: '[Eclipse] RCP實戰'
date: 2024-05-09 13:10:29
tags:
- Eclipse
- RCP
---

實務上遇到的專案有用到自訂的taglib和規則，為開發方便且好追code，希望按下Ctrl鍵後，便能快速顯示超連結，且能跳到指定的類別和方法，因此有自行寫Eclipse外掛的想法。
<!--more-->
# 修改plugin.xml

建好Plug-in Project後，修改plugin.xml如下
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?eclipse version="3.4"?>
<plugin>
    <extension point="org.eclipse.ui.workbench.texteditor.hyperlinkDetectors">
        <hyperlinkDetector
            class="com.usermark.plugin.detectors.HyperlinkDetector"
            id="com.usermark.hyperlinkDetector"
            name="Custom Link"
            targetId="org.eclipse.ui.DefaultTextEditor">
        </hyperlinkDetector>
    </extension>
</plugin>
```
其中，第4行point屬性為org.eclipse.ui.workbench.texteditor.hyperlinkDetectors，表示針對超連結偵測做擴展。切記name屬性必須要有，否則執行時無法成功。

# 實作AbstractHyperlinkDetector

接著建立HyperlinkDetector類別來實作，其靈感來自於XMLJavaHyperlinkDetector，有興趣可以自行查看。
```java
public class HyperlinkDetector extends AbstractHyperlinkDetector {
    @Override
    public IHyperlink[] detectHyperlinks(ITextViewer textViewer, IRegion region, boolean canShowMultipleHyperlinks) {
        if (region != null && textViewer != null) {
            IDocument document = textViewer.getDocument();
            ITextFileBuffer textFileBuffer = FileBuffers.getTextFileBufferManager().getTextFileBuffer(document);
            if (textFileBuffer != null) {
                IPath basePath = textFileBuffer.getLocation();
                if (basePath != null && !basePath.isEmpty()) {
                    IProject project = ResourcesPlugin.getWorkspace().getRoot().getProject(basePath.segment(0));
                }
            }
        }

        return null;
    }
}
```
經過一系列操作，第10行便可取得專案類別IProject，這在後面會用到。

# IRegion

那麼當游標停在某個code，未做圈選，如何得知哪個範圍的code要來判斷是否產生超連結呢？
答案是利用detectHyperlinks()第二個傳入參數region得知游標停在哪，搭配第5行的document前後移動游標，便能依自訂規則取出字串，實作selectQualifiedName()方法如下。
```java
private NodeRegion selectQualifiedName(IDocument document, int anchor) {
    try {
        int offset = anchor;

        while (offset >= 0) {
            char c = document.getChar(offset);
            if (!Character.isJavaIdentifierPart(c) && c != '.')
                break;
            offset--;
        }

        int start = offset;

        offset = anchor;
        int length = document.getLength();

        while (offset < length) {
            char c = document.getChar(offset);
            if (!Character.isJavaIdentifierPart(c) && c != '.')
                break;
            offset++;
        }

        int end = offset;

        if (start == end) {
            return null;
        }
        
        // 往前找屬性名稱
        offset = start;
        int attrEnd = start;
        
        while (offset >= 0) {
            char c = document.getChar(offset);
            if (c == '=')
                attrEnd = offset;
            else if (c == ' ')
                break;
            offset--;
        }
        
        int attrStart = offset;
        String attrName = document.get(attrStart + 1, attrEnd - attrStart - 1);
        
        // 往前找Tag名稱
        while (offset >= 0) {
            char c = document.getChar(offset);
            if (c == '<')
                break;
            offset--;
        }
        
        int tagStart = offset;
        
        while (offset < attrStart) {
            char c = document.getChar(offset);
            if (c == ' ')
                break;
            offset++;
        }

        int tagEnd = offset;
        String tagName = document.get(tagStart + 1, tagEnd - tagStart - 1);
        
        return new NodeRegion(start + 1, end - start - 1, tagName, attrName);

    } catch (BadLocationException badLocationException) {
        return null;
    }
}
```

# 產生超連結

此處為方便說明，只呈現關鍵code，主要就是丟給createHyperlink()。
```java
IPath basePath = textFileBuffer.getLocation();
IProject project = ResourcesPlugin.getWorkspace().getRoot().getProject(basePath.segment(0));
NodeRegion hyperlinkRegion = selectQualifiedName(document, region.getOffset());
List<IHyperlink> links = createHyperlink(project, basePath, name, hyperlinkRegion);
```

```java
private List<IHyperlink> createHyperlink(IProject project, IPath basePath, String elementName, NodeRegion region) {
    IJavaProject javaProject = JavaCore.create(project);
    if (javaProject != null && javaProject.exists()) {
        try {
            if (region.getTagName().equals("my:select") && region.getAttrName().equals("src")) {
                IType element = javaProject.findType("your/full/class");
                if (element != null && element.exists()) {
                    return Collections.singletonList(new JavaElementHyperlink(region, element));
                }
            } else if (region.getTagName().equals("my:button") && region.getAttrName().equals("action")) {
                // 自行實作
            }
        } catch (Exception e) {
        }
    }

    return null;
}
```

舉例有個html元素長這樣
```html
<my:select src="com.usermark.tag.selectImp" />
```
當游標停在src的value時，便可呈現超連結。
那如果判斷過程中需要搭配xml配置的話也辦得到。

```java
DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = builderFactory.newDocumentBuilder();
Document xmlDocument = builder.parse(project.getFile("your/xml/path").getContents());
XPath xPath = XPathFactory.newInstance().newXPath();
String expression = "your/expression";
Node controller = (Node) xPath.compile(expression).evaluate(xmlDocument, XPathConstants.NODE);
```

或是使用

# 實作IHyperlink

透過JavaUI.openInEditor()開啟IJavaElement。

```java
static class JavaElementHyperlink implements IHyperlink {
    private final IJavaElement fElement;
    private final IRegion fRegion;

    JavaElementHyperlink(IRegion region, IJavaElement element) {
        this.fRegion = region;
        this.fElement = element;
    }

    @Override
    public IRegion getHyperlinkRegion() {
        return this.fRegion;
    }

    // 可以完整顯示method + class
    @Override
    public String getHyperlinkText() {
        String elementLabel = JavaElementLabels.getElementLabel(this.fElement, 
                JavaElementLabels.M_POST_QUALIFIED 
                | JavaElementLabels.I_POST_QUALIFIED 
                | JavaElementLabels.F_POST_QUALIFIED
                | JavaElementLabels.T_POST_QUALIFIED
                | JavaElementLabels.TP_POST_QUALIFIED
                | JavaElementLabels.D_POST_QUALIFIED
                | JavaElementLabels.CF_POST_QUALIFIED
                | JavaElementLabels.CU_POST_QUALIFIED
                | JavaElementLabels.P_POST_QUALIFIED
                | JavaElementLabels.ROOT_VARIABLE
                | JavaElementLabels.ROOT_POST_QUALIFIED);
        return NLS.bind(JSPUIMessages.Open, elementLabel);
    }

    @Override
    public String getTypeLabel() {
        return null;
    }

    @Override
    public void open() {
        try {
            JavaUI.openInEditor(this.fElement);
        } catch (PartInitException partInitException) {
        } catch (JavaModelException javaModelException) {
        }
    }
}
```

# 完整code

```java
package com.usermark.plugin.detectors;

import java.lang.reflect.Array;
import java.util.Collections;
import java.util.List;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.xpath.XPath;
import javax.xml.xpath.XPathConstants;
import javax.xml.xpath.XPathFactory;

import org.eclipse.core.filebuffers.FileBuffers;
import org.eclipse.core.filebuffers.ITextFileBuffer;
import org.eclipse.core.resources.IProject;
import org.eclipse.core.resources.ResourcesPlugin;
import org.eclipse.core.runtime.CoreException;
import org.eclipse.core.runtime.IPath;
import org.eclipse.jdt.core.IJavaElement;
import org.eclipse.jdt.core.IJavaProject;
import org.eclipse.jdt.core.IType;
import org.eclipse.jdt.core.JavaCore;
import org.eclipse.jdt.core.JavaModelException;
import org.eclipse.jdt.ui.JavaElementLabels;
import org.eclipse.jdt.ui.JavaUI;
import org.eclipse.jface.text.BadLocationException;
import org.eclipse.jface.text.IDocument;
import org.eclipse.jface.text.IRegion;
import org.eclipse.jface.text.ITextViewer;
import org.eclipse.jface.text.Region;
import org.eclipse.jface.text.hyperlink.AbstractHyperlinkDetector;
import org.eclipse.jface.text.hyperlink.IHyperlink;
import org.eclipse.jst.jsp.core.internal.java.JSPTranslation;
import org.eclipse.jst.jsp.core.internal.java.JSPTranslationAdapter;
import org.eclipse.jst.jsp.core.internal.java.JSPTranslationExtension;
import org.eclipse.jst.jsp.ui.internal.JSPUIMessages;
import org.eclipse.osgi.util.NLS;
import org.eclipse.ui.PartInitException;
import org.eclipse.wst.sse.core.StructuredModelManager;
import org.eclipse.wst.sse.core.internal.provisional.IStructuredModel;
import org.eclipse.wst.xml.core.internal.provisional.document.IDOMDocument;
import org.eclipse.wst.xml.core.internal.provisional.document.IDOMModel;
import org.w3c.dom.Document;
import org.w3c.dom.Node;

public class HyperlinkDetector extends AbstractHyperlinkDetector {
    
    static class NodeRegion extends Region {
        private String tagName;
        private String attrName;

        public NodeRegion(int offset, int length, String tagName, String attrName) {
            super(offset, length);
            this.tagName = tagName;
            this.attrName = attrName;
        }
        
        public String getTagName() {
            return tagName;
        }
        
        public String getAttrName() {
            return attrName;
        }
    }

    static class JavaElementHyperlink implements IHyperlink {
        private final IJavaElement fElement;
        private final IRegion fRegion;

        JavaElementHyperlink(IRegion region, IJavaElement element) {
            this.fRegion = region;
            this.fElement = element;
        }

        @Override
        public IRegion getHyperlinkRegion() {
            return this.fRegion;
        }

        // 可以完整顯示method + class
        @Override
        public String getHyperlinkText() {
            String elementLabel = JavaElementLabels.getElementLabel(this.fElement, 
                    JavaElementLabels.M_POST_QUALIFIED 
                    | JavaElementLabels.I_POST_QUALIFIED 
                    | JavaElementLabels.F_POST_QUALIFIED
                    | JavaElementLabels.T_POST_QUALIFIED
                    | JavaElementLabels.TP_POST_QUALIFIED
                    | JavaElementLabels.D_POST_QUALIFIED
                    | JavaElementLabels.CF_POST_QUALIFIED
                    | JavaElementLabels.CU_POST_QUALIFIED
                    | JavaElementLabels.P_POST_QUALIFIED
                    | JavaElementLabels.ROOT_VARIABLE
                    | JavaElementLabels.ROOT_POST_QUALIFIED);
            return NLS.bind(JSPUIMessages.Open, elementLabel);
        }

        @Override
        public String getTypeLabel() {
            return null;
        }

        @Override
        public void open() {
            try {
                JavaUI.openInEditor(this.fElement);
            } catch (PartInitException partInitException) {
            } catch (JavaModelException javaModelException) {
            }
        }
    }

    private boolean isJspJavaContent(IDocument document, IRegion region) {
        JSPTranslation translation = null;
        IStructuredModel model = null;
        try {
            model = StructuredModelManager.getModelManager().getExistingModelForRead(document);
            if (model instanceof IDOMModel) {
                IDOMModel xmlModel = (IDOMModel) model;
                if (xmlModel != null) {
                    IDOMDocument xmlDoc = xmlModel.getDocument();
                    JSPTranslationAdapter adapter = (JSPTranslationAdapter) xmlDoc
                            .getAdapterFor(org.eclipse.jst.jsp.core.internal.java.IJSPTranslation.class);
                    if (adapter != null) {
                        JSPTranslationExtension jSPTranslationExtension = adapter.getJSPTranslation();
                        if (jSPTranslationExtension != null) {
                            int javaOffset = jSPTranslationExtension.getJavaOffset(region.getOffset());
                            if (javaOffset > -1) {
                                return true;
                            }
                        }
                    }
                }
            }
        } finally {
            if (model != null) {
                model.releaseFromRead();
            }
        }

        return false;
    }

    private List<IHyperlink> createHyperlink(IProject project, IPath basePath, String elementName, NodeRegion region) {
        IJavaProject javaProject = JavaCore.create(project);
        if (javaProject != null && javaProject.exists()) {
            try {
                if (region.getTagName().equals("my:select") && region.getAttrName().equals("src")) {
                    IType element = javaProject.findType("your/full/class");
                    if (element != null && element.exists()) {
                        return Collections.singletonList(new JavaElementHyperlink(region, element));
                    }
                } else if (region.getTagName().equals("my:button") && region.getAttrName().equals("action")) {
                    DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
                    DocumentBuilder builder = builderFactory.newDocumentBuilder();
                    Document xmlDocument = builder.parse(project.getFile("your/xml/path").getContents());
                    XPath xPath = XPathFactory.newInstance().newXPath();
                    String expression = "your/expression";
                    Node controller = (Node) xPath.compile(expression).evaluate(xmlDocument, XPathConstants.NODE);
                    // 自行實作
                }
            } catch (Exception e) {
            }
        }

        return null;
    }

    @Override
    public IHyperlink[] detectHyperlinks(ITextViewer textViewer, IRegion region, boolean canShowMultipleHyperlinks) {
        if (region != null && textViewer != null) {
            IDocument document = textViewer.getDocument();
            ITextFileBuffer textFileBuffer = FileBuffers.getTextFileBufferManager().getTextFileBuffer(document);
            if (textFileBuffer != null) {
                IPath basePath = textFileBuffer.getLocation();
                if (basePath != null && !basePath.isEmpty()) {
                    IProject project = ResourcesPlugin.getWorkspace().getRoot().getProject(basePath.segment(0));
                    try {
                        if (basePath.segmentCount() > 1 && project.isAccessible()
                                && project.hasNature(JavaCore.NATURE_ID)) {

                            NodeRegion hyperlinkRegion = selectQualifiedName(document, region.getOffset());
                            if (hyperlinkRegion == null || isJspJavaContent(document, hyperlinkRegion)) {
                                return null;
                            }
                            String name = null;
                            try {
                                name = document.get(hyperlinkRegion.getOffset(), hyperlinkRegion.getLength()).trim();
                            } catch (BadLocationException badLocationException) {
                            }

                            if (name != null && !name.isEmpty()) {
                                List<IHyperlink> links = createHyperlink(project, basePath, name, hyperlinkRegion);
                                if (links != null) {
                                    Object array = Array.newInstance(IHyperlink.class, links.size());
                                    return (IHyperlink[]) links.toArray((Object[])array);
                                }
                            }
                        }
                    } catch (CoreException e) {
                    }
                }
            }
        }

        return null;
    }

    private NodeRegion selectQualifiedName(IDocument document, int anchor) {
        try {
            int offset = anchor;

            while (offset >= 0) {
                char c = document.getChar(offset);
                if (!Character.isJavaIdentifierPart(c) && c != '.')
                    break;
                offset--;
            }

            int start = offset;

            offset = anchor;
            int length = document.getLength();

            while (offset < length) {
                char c = document.getChar(offset);
                if (!Character.isJavaIdentifierPart(c) && c != '.')
                    break;
                offset++;
            }

            int end = offset;

            if (start == end) {
                return null;
            }
            
            // 往前找屬性名稱
            offset = start;
            int attrEnd = start;
            
            while (offset >= 0) {
                char c = document.getChar(offset);
                if (c == '=')
                    attrEnd = offset;
                else if (c == ' ')
                    break;
                offset--;
            }
            
            int attrStart = offset;
            String attrName = document.get(attrStart + 1, attrEnd - attrStart - 1);
            
            // 往前找Tag名稱
            while (offset >= 0) {
                char c = document.getChar(offset);
                if (c == '<')
                    break;
                offset--;
            }
            
            int tagStart = offset;
            
            while (offset < attrStart) {
                char c = document.getChar(offset);
                if (c == ' ')
                    break;
                offset++;
            }

            int tagEnd = offset;
            String tagName = document.get(tagStart + 1, tagEnd - tagStart - 1);
            
            return new NodeRegion(start + 1, end - start - 1, tagName, attrName);

        } catch (BadLocationException badLocationException) {
            return null;
        }
    }
}
```