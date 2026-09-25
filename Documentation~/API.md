# com.vahtyah.core — API Reference

> Version 1.0.0. Auto-generated from the compiled public API by `VahTyah → Generate API Docs`. Do not edit by hand — edit the source or `Documentation~/Examples.md` instead.

This package ships as obfuscated DLLs. This file documents the full public surface so it can be used without decompiling.

## Usage Examples

Editor-only utilities shared by the other VahTyah packages: asset creation/lookup helpers
(`EditorUtils`) and a layered box renderer (`LayerDrawingSystem` + `LayerConfiguration`) used to
draw rounded backgrounds, borders and gradients in custom Inspectors/EditorWindows. Everything here
lives in the `VahTyah.Core.Editor` assembly, so use it from editor code only.

### Asset helpers (`EditorUtils`)

```csharp
using UnityEditor;
using UnityEngine;
using VahTyah.Core;

// Create a ScriptableObject asset at an explicit path.
var db = EditorUtils.CreateAsset<MyDatabase>("Assets/_Project/Data/MyDatabase.asset", refresh: true);

// Or create one by name into a conventional location.
var settings = EditorUtils.CreateScriptableObject<MySettings>("MySettings");

// Find existing assets in the project (by type, optionally filtered by name).
MyDatabase found = EditorUtils.GetAsset<MyDatabase>();          // first of type
MyDatabase byName = EditorUtils.GetAssetByName<MyDatabase>("MyDatabase");
MyDatabase[] all = EditorUtils.GetAssets<MyDatabase>();
bool exists = EditorUtils.HasAsset<MyDatabase>();

// Misc project helpers.
string rel = EditorUtils.ConvertToRelativePath(absolutePath);  // "Assets/..."
string folder = EditorUtils.FindFolderPath("Data");
EditorUtils.SelectAsset(found);                                 // ping/select in Project
```

### Draw a layered box in a custom editor (`LayerDrawingSystem`)

`LayerConfiguration` is a stack of `Layer`s (solid color, rounded rect, border, gradient) drawn back
to front into a `Rect`. Use the factory methods rather than building layers by hand.

```csharp
using UnityEditor;
using UnityEngine;
using VahTyah.Core;

public class MyWindow : EditorWindow
{
    private readonly LayerConfiguration _card =
        LayerConfiguration.CreateCardStyle(
            cardColor:   new Color(0.20f, 0.20f, 0.22f),
            shadowColor: new Color(0f, 0f, 0f, 0.35f),
            cornerRadius: 6f);

    private void OnGUI()
    {
        Rect box = GUILayoutUtility.GetRect(0, 80, GUILayout.ExpandWidth(true));
        LayerDrawingSystem.DrawLayers(box, _card);   // no-op outside Repaint, safe to call every frame
        // ... draw your controls inside `box`
    }
}
```

Other ready-made configurations: `LayerConfiguration.CreateSimpleBackground(color)`,
`CreateBackgroundWithBorder(bg, border, borderWidth, borderRadius)`, and single `Layer` factories
(`Layer.CreateSolidColor`, `CreateRoundedRect`, `CreateBorder`, `CreateGradient`,
`CreateRoundedGradient`) which you can push into a `LayerConfiguration` for custom stacks.

## API Reference

### namespace `VahTyah.Core`

#### static class `EditorUtils`

Editor-only helpers for finding, creating, and selecting project assets, plus small SerializedProperty/path utilities shared across the VahTyah editor tooling.  

```csharp
public static readonly string projectFolderPath;
public static string ConvertToRelativePath(string absolutePath);
public static T CreateAsset<T>(string path, bool refresh = false);
public static T CreateAsset<T>(Type type, string path, bool refresh = false);
public static T CreateScriptableObject<T>(string assetName);
public static string FindFolderPath(string folderName);
public static Object GetAsset(Type type);
public static T GetAsset<T>(string name = "");
public static T GetAssetByName<T>(string name = "");
public static T[] GetAssets<T>(string name = "");
public static GenericMenu GetSubTypeMenu(Type parentType, Action<Type> selectAction, Type selectedType = null, bool showAbstract = false);
public static bool HasAsset<T>(string name = "");
public static bool IsArray(SerializedProperty property);
public static bool IsArray(string propertyPath);
public static void SelectAsset(SerializedProperty serializedProperty);
public static void SelectAsset(Object objectReference);
```

- `projectFolderPath` — Absolute path to the project root (the folder containing Assets/), with a trailing slash.
- `GetSubTypeMenu` — Build a GenericMenu listing a type and all its subclasses (nested by inheritance depth), calling selectAction with the chosen type. selectedType is shown checked; abstract types are included only when showAbstract is true.
- `SelectAsset` — Focus the Project window and select the asset referenced by the property (no-op if null).
- `GetAsset` — Get asset in project
- `GetAssetByName` — Get asset in project
- `GetAssets` — Get assets in project
- `HasAsset` — Check if project contains asset
- `CreateAsset` — Create ScriptableObject at path
- `CreateScriptableObject` — Create a ScriptableObject asset next to the current Project selection (or under Assets/ if nothing is selected), auto-numbering the file name to avoid overwriting, then select it.
- `FindFolderPath` — Find the first folder with this name anywhere under Assets/; returns its path, or empty (logging a warning) if not found.
- `IsArray` — True if the property is an element inside a serialized array/list (path contains "Array.data").
- `ConvertToRelativePath` — Convert an absolute file path to a project-relative "Assets/..." path (forward slashes).

#### enum `GradientDirection`

Direction of a gradient layer's color ramp.  

```csharp
enum GradientDirection : int
{
    Horizontal = 0,
    Vertical = 1,
}
```

#### interface `IEditorPanel`

A tabbed panel hosted by PanelNavigator. Implement to add a named tab to the editor content area.  

```csharp
public abstract void Draw(Rect rect);
public abstract void Initialize();
public abstract void OnDisable();
public abstract void OnEnable();
```

#### class `Layer`

One drawable layer of a LayerConfiguration — a solid/rounded/border/gradient fill with per-side Padding and per-corner border width/radius. Build with the Create* factories.  

```csharp
public Layer();
public Vector4 borderRadius;
public Vector4 borderWidth;
public Color color;
public bool enabled;
public GradientDirection gradientDirection;
public Color gradientEndColor;
public Padding padding;
public LayerType type;
public Layer Clone();
public static Layer CreateBorder(Color color, float borderWidth = 1, float borderRadius = 0, Padding padding = null);
public static Layer CreateGradient(Color startColor, Color endColor, GradientDirection direction = GradientDirection.Vertical, Padding padding = null);
public static Layer CreateRoundedGradient(Color startColor, Color endColor, float borderRadius = 4, GradientDirection direction = GradientDirection.Vertical, Padding padding = null);
public static Layer CreateRoundedRect(Color color, float borderRadius = 4, Padding padding = null);
public static Layer CreateSolidColor(Color color, Padding padding = null);
```

- `borderWidth` — Border thickness, as the Vector4 passed to GUI.DrawTexture (Unity's edge order). Set uniformly by the Create* factories.
- `borderRadius` — Corner radius, as the Vector4 passed to GUI.DrawTexture (Unity's corner order). Set uniformly by the Create* factories.
- `CreateSolidColor` — A flat solid-color fill.
- `CreateBorder` — An outline of uniform width and corner radius (no fill).
- `CreateRoundedRect` — A solid fill with rounded corners.
- `CreateGradient` — A two-color linear gradient (horizontal or vertical).
- `CreateRoundedGradient` — A two-color gradient with rounded corners.
- `Clone` — Deep copy of this layer (padding cloned too).

#### class `LayerConfiguration`

An ordered stack of Layers drawn back-to-front by LayerDrawingSystem to build a box background (solid fills, rounded rects, borders, gradients). Use the Create* factories for common looks.  

```csharp
public LayerConfiguration();
public LayerConfiguration(int layerCount);
public Layer[] layers;
public void AddLayer(Layer layer);
public LayerConfiguration Clone();
public static LayerConfiguration CreateBackgroundWithBorder(Color backgroundColor, Color borderColor, float borderWidth = 1, float borderRadius = 0);
public static LayerConfiguration CreateCardStyle(Color cardColor, Color shadowColor, float cornerRadius = 4);
public static LayerConfiguration CreateSimpleBackground(Color color);
public Layer GetLayerByType(LayerType type);
```

- `layers` — The layers, painted in array order — index 0 first (bottom), the last on top.
- `AddLayer` — Append a layer on top of the stack (grows the array by one).
- `GetLayerByType` — Return the first layer of the given type, or null if none.
- `Clone` — Deep copy — clones every layer so the copy can be edited independently.
- `CreateSimpleBackground` — One solid-color fill layer.
- `CreateBackgroundWithBorder` — A rounded fill with a border layer on top.
- `CreateCardStyle` — A card look: a rounded shadow layer offset behind a rounded fill.

#### static class `LayerDrawingSystem`

Renders a LayerConfiguration into a Rect for custom Editors/EditorWindows (IMGUI).  

```csharp
public static void DrawLayers(Rect targetRect, LayerConfiguration config);
```

- `DrawLayers` — Draw the configuration's enabled layers into targetRect (bottom to top). No-op outside the Repaint event, so it is safe to call every OnGUI frame.

#### enum `LayerType`

The kind of fill a Layer draws.  

```csharp
enum LayerType : int
{
    SolidColor = 0,
    RoundedRect = 1,
    Border = 2,
    Gradient = 3,
    RoundedGradient = 4,
}
```

#### class `Padding`

Per-side inset (left/right/top/bottom) that shrinks a Layer's rect before it is drawn. Constructors: (), (all), (horizontal, vertical), (left, right, top, bottom).  

```csharp
public Padding();
public Padding(float all);
public Padding(float horizontal, float vertical);
public Padding(float left, float right, float top, float bottom);
public float bottom;
public float left;
public float right;
public float top;
```

#### class `PanelNavigator`

Draws a tab bar for a set of named IEditorPanels and renders the active one. The tab-bar/content styling comes from an injected PanelNavigatorStyle; when none is given it falls back to PanelNavigatorTheme — the per-user theme stored in UserSettings/.  

```csharp
public PanelNavigator(Dictionary<string, IEditorPanel> panels, PanelNavigatorStyle style = null);
public void Cleanup();
public void Draw(Rect rect);
public void OnEnable();
```

- `OnEnable` — Forward the enable signal to every hosted panel.
- `Cleanup` — Forward the disable signal to every hosted panel (call on window disable).
- `Draw` — Draw the active panel content and the tab bar within the given rect.

#### class `PanelNavigatorStyle`

Visual style for a PanelNavigator: the tab-bar height, the active/inactive tab layer stacks, and the content-area background. Plain serializable data (no ScriptableObject), so it can be embedded, injected, or wrapped by PanelNavigatorTheme.  

```csharp
public PanelNavigatorStyle();
public LayerConfiguration activeTab;
public LayerConfiguration contentBackground;
public LayerConfiguration inactiveTab;
public float menuBarHeight;
public float ContentBorderWidth { get; }
public static PanelNavigatorStyle CreateDefault(bool isDark);
```

- `ContentBorderWidth` — Border width used for the active-tab overlap trick (the active tab grows down to sit on top of the content border). Reads the content background's border layer; safe when it has fewer layers.
- `CreateDefault` — Build the built-in style for the given skin. Values mirror the original Level Editor defaults.

