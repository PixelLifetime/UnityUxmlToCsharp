# Unity Uxml To Csharp

Plugin that generates helper C# classes based on uxml files in Unity. It may be considered as a very simple and automatic alternative to [UxmlFactory](https://docs.unity3d.com/Manual/UIE-UXML.html).

## Installation:
**Easy way:**\
Add `"com.ks.uxmltocsharp": "https://github.com/kamyker/UnityUxmlToCsharp.git"` to `<your_project>/Packages/manifest.json` dependencies\
**Git way:**\
Clone repo as submodule to `<your_project>/Packages/UxmlToCsharp`

## Quick example
Let's make a panel with 3 buttons. Their actions have to be assigned in C#. Additional we will hide them when window is too small.

### Normal way:
**Page.uxml**
```
<?xml version="1.0" encoding="utf-8"?>
<UXML
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="UnityEngine.UIElements"
    xsi:noNamespaceSchemaLocation="../UIElementsSchema/UIElements.xsd"
    xsi:schemaLocation="UnityEngine.UIElements ../UIElementsSchema/UnityEngine.UIElements.xsd">
  <Box>
      <Button name="testButton1"/>
      <Button name="testButton2"/>
      <Button name="testButton3"/>
  </Box>
</UXML>
```
**TestWindow.cs**
```
public class TestWindow : EditorWindow
{
    private Button testButton1;
    private Button testButton2;
    private Button testButton3;
    
    private void OnEnable()
    {
        var root = rootVisualElement;
        root.Clear();
        var template = AssetDatabase.LoadAssetAtPath<VisualTreeAsset>("page.uxml"); //change this to correct path
        template.CloneTree(root);
        testButton1 = root.Q<Button>("testButton1");
        testButton2 = root.Q<Button>("testButton2");
        testButton3 = root.Q<Button>("testButton3");
        testButton1.clickable.clicked += () => Debug.Log("button1 clicked");
        testButton2.clickable.clicked += () => Debug.Log("button2 clicked");
        testButton3.clickable.clicked += () => Debug.Log("button3 clicked");
        
        root.RegisterCallback<GeometryChangedEvent>(OnWindowResized);
    }
    
    private void OnWindowResized(GeometryChangedEvent window)
    {
        //using testButton1, testButton2, testButton3 here
    }
}
```

### With this plugin:
Uxml looks the same but we change 3 lines with buttons and their names to custom style "--csName".
```
<Button style="--csName: TestButton1;"/>
<Button style="--csName: TestButton2;"/>
<Button style="--csName: TestButton3;"/>
```
After creating UXML right click it in Unity and select "Uxml To C#/Create C# class". New file called PageConverted.cs will be generated.

**TestWindow.cs**
```
public class TestWindow : EditorWindow
{
    private PageConverted page;
    
    private void OnEnable()
    {
        var root = rootVisualElement;
        root.Clear();
        page = new PageConverted();
        root.AddChildrenOf(page); //extension method similar to template.CloneTree(root);
        page.TestButton1.clickable.clicked += () => Debug.Log("button1 clicked");
        page.TestButton2.clickable.clicked += () => Debug.Log("button2 clicked");
        page.TestButton3.clickable.clicked += () => Debug.Log("button3 clicked");
        
        root.RegisterCallback<GeometryChangedEvent>(OnWindowResized);
    }
    
    private void OnWindowResized(GeometryChangedEvent window)
    {
        //using page.TestButton1, page.TestButton2, page.TestButton3 here
    }
}
```

As you can see you no longer have to use Q method and load specific uxml template as it all happens in `new PageConverted()`.

## Important
1. Converted class will be automatically updated every time uxml is changed IF they are both placed in the same folder.
2. If above is not an option and you want to move it to different folder you can manually update generated class with RMB and "Uxml To C#/Update C# class". It works as class has hardcoded guid to uxml file.

# Templates
Have you read about [UXML Templates](https://docs.unity3d.com/Manual/UIE-WritingUXMLTemplate.html)? This project supports something similar. Simply add `"style="--csTemplate: <name-of-converted-uxml>;"` custom style. Whole object with this parameter will be replaced with the template. What's interesting is that you can extend your converted uxml class, add custom logic and still use it the same way! This makes parts of your UI properly decoupled.

### Example:
We are going to convert PageConverted to standalone solution and show it 3 times instead of 1 like in previous example.
**Page.uxml**\
This file stays the same, remember to also generate C# class (like in the example above)
```
<?xml version="1.0" encoding="utf-8"?>
<UXML
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="UnityEngine.UIElements"
    xsi:noNamespaceSchemaLocation="../UIElementsSchema/UIElements.xsd"
    xsi:schemaLocation="UnityEngine.UIElements ../UIElementsSchema/UnityEngine.UIElements.xsd">
  <Box>
    <Button style="--csName: TestButton1;"/>
    <Button style="--csName: TestButton2;"/>
    <Button style="--csName: TestButton3;"/>
  </Box>
</UXML>
```
**PageConvertedExtended.cs**\
Here we are going to add logic to our converted uxml.
```
public class PageConvertedExtended : PageConverted
{
    private void PageConvertedExtended() //PageConverted constructor is called before this
    {
        TestButton1.clickable.clicked += () => Debug.Log("button1 clicked");
        TestButton2.clickable.clicked += () => Debug.Log("button2 clicked");
        TestButton3.clickable.clicked += () => Debug.Log("button3 clicked");
        
        TestButton1.RegisterCallback<GeometryChangedEvent>(OnWindowResized);
    }
    
    private void OnWindowResized(GeometryChangedEvent window)
    {
        //using TestButton1, TestButton2, TestButton3 here
    }
}
```
**TriplePages.uxml**\
Page with templates
```
<?xml version="1.0" encoding="utf-8"?>
<UXML
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="UnityEngine.UIElements"
    xsi:noNamespaceSchemaLocation="../UIElementsSchema/UIElements.xsd"
    xsi:schemaLocation="UnityEngine.UIElements ../UIElementsSchema/UnityEngine.UIElements.xsd">
  <Box style="--csTemplate: PageConvertedExtended;">
  <Box style="--csTemplate: PageConvertedExtended;">
  <Box style="--csTemplate: PageConvertedExtended;">
</UXML>
```
Create C# class from this uxml and it's done. Now you can add TriplePagesConverted or PageConvertedExtended to whatever you want.\
**TestWindow.cs**
```
public class TestWindow : EditorWindow
{
    private void OnEnable()
    {
        var root = rootVisualElement;
        root.Clear();
        root.AddChildrenOf(new TriplePagesConverted());
    }
    
    [MenuItem("TestWindow/Open")]
    private static void Open()
    {
        var window = GetWindow<TestWindow>();
        window.Show();
    }
}
```
Here's our final result:
![window](/Documentation~/img.png)
