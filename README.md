# Editors

Editors are used to allow Editing of Assets. Unity provides a lot of Default Editors already that simplify editing your Assets. Without Editors, your Designers would have to edit TextFiles all the time.

## Serialized Fields

For any class inheriting from `UnityEngin.Object`, the Inspector shows 

- Public Fields without `[HideInInspector]`
- `private`/`protected` Fields with `[SerializeField]`

## Attributes

### MonoBehaviors

`[SelectionBase]`

This makes it much easier to select your objects in the Scene View. Clicks on visible elements in the Scene which are parented underneath (e.g. Texts) will now Select the Parent with this Attribute instead.

`[ContextMenu("MethodName")]`

Add a Context-Menu (Rightclick-Action) to a MonoBehavior by placing this Attribute over a Monobehavior.

`[AddComponentMenu("FightSystem/Attack")]`

Change the Path of the Component in the Component Menu (when using "Add Component") by placing this Attribute over a Monobehavior.

`[CreateAssetMenu]`

Put over ScriptableObjects to create an easy way of creating an Asset Instance in the Project Window.

`[RequireComponent(typeof(OtherComponent))]`

Make sure that another Component Exists on the same GameObject, if your Component depends on it.

`[ExecuteAlways]`

This will enable your Script's EventMethods like `Update`, `OnGUI` and `OnRenderObject` to be called always (Game, Editor, Prefab Editor)

`[ExecuteInEditMode]`

This will enable your Script's EventMethods like `Update`, `OnGUI` and `OnRenderObject` to be called in Game and Editor, but not in Prefab Editor.

`[Icon("path/to/icon.png")]`

Add a Custom Icon to your Component.

`[HelpUrl("path/to/documentation")]`

Add Documentation to your Component (The Questionmark Icon in the Inspector)

### Fields

`[Header("Settings")]`

Group your Inspector Values visibly using this Attribute.

`[Tooltip("This field controls, how...")]`

Provide some additional information in a tooltip by placing this Attribute over a Field.

`[Range(0,1)]`

Limit numeric Values to values between both provided numbers by placing this Attribute over a Field.

`[TextArea]`

Makes a Text Editable in an Area underneath the Field Name. Use for very long texts by placing this Attribute over a Field.

`[Multiline]`

Makes a Text Editable in an Area next to the Field Name. Use for medium-length texts by placing this Attribute over a Field.

`[ContextMenuItem("Visible Name", "MethodName"]`

Add a Context-Menu (Rightclick-Action) to a Field by placing this Attribute over a Field.

`[Delayed]`

Makes the Input on a Field be applied with a small Delay. Very useful, if temporary changes could break your Game (e.g. if `0` crashes your game, but somesome wants to type `0.3`

`[InspectorName]`

Changes the name that is used for your Field in the Inspector.

`[Min(0)]`

Requires a numeric value to have at least the given value. A good way to prevent negative numbers.

`[SerializeReference]`

This allows you to serialize classes as References rather than values. Allows Polymorphism and Nested Classes.

### Other

`[InspectorOrder(InspectorSort.ByValue, InspectorSortDirection.Descending)]`

Control, in what Order Enum Values are Sorted in. Sometimes, it makes more sence by Value, sometimes Alphabetically.

## Custom Property

```cs
// This is not an editor script. The property attribute class should be placed in a regular script file.
using UnityEngine;

public class RangeAttribute : PropertyAttribute
{
    public float min;
    public float max;

    public RangeAttribute(float min, float max)
    {
        this.min = min;
        this.max = max;
    }
}
```

```cs
// The property drawer class should be placed in an editor script, inside a folder called Editor.

// Tell the RangeDrawer that it is a drawer for properties with the RangeAttribute.
using UnityEngine;
using UnityEditor;

[CustomPropertyDrawer(typeof(RangeAttribute))]
public class RangeDrawer : PropertyDrawer
{
    // Draw the property inside the given rect
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
        // First get the attribute since it contains the range for the slider
        RangeAttribute range = attribute as RangeAttribute;

        // Now draw the property as a Slider or an IntSlider based on whether it's a float or integer.
        if (property.propertyType == SerializedPropertyType.Float)
            EditorGUI.Slider(position, property, range.min, range.max, label);
        else if (property.propertyType == SerializedPropertyType.Integer)
            EditorGUI.IntSlider(position, property, Convert.ToInt32(range.min), Convert.ToInt32(range.max), label);
        else
            EditorGUI.LabelField(position, label.text, "Use Range with float or int.");
    }
}
```

## Custom Inspector (MonoBehaviour)

```cs
using UnityEngine;
[ExecuteInEditMode]
public class LookAtPoint : MonoBehaviour
{
    public Vector3 lookAtPoint = Vector3.zero;

    void Update()
    {
        transform.LookAt(lookAtPoint);
    }
}
```

```cs
using UnityEngine;
using UnityEditor;

[CustomEditor(typeof(LookAtPoint))]
[CanEditMultipleObjects]
public class LookAtPointEditor : Editor
{
    SerializedProperty lookAtPoint;

    void OnEnable()
    {
        lookAtPoint = serializedObject.FindProperty("lookAtPoint");
    }

    public override void OnInspectorGUI()
    {
        serializedObject.Update();
        EditorGUILayout.PropertyField(lookAtPoint);
        if (lookAtPoint.vector3Value.y > (target as LookAtPoint).transform.position.y)
        {
            EditorGUILayout.LabelField("(Above this object)");
        }
        if (lookAtPoint.vector3Value.y < (target as LookAtPoint).transform.position.y)
        {
            EditorGUILayout.LabelField("(Below this object)");
        }
        
            
        serializedObject.ApplyModifiedProperties();
    }

    public void OnSceneGUI()
    {
        var t = (target as LookAtPoint);

        EditorGUI.BeginChangeCheck();
        Vector3 pos = Handles.PositionHandle(t.lookAtPoint, Quaternion.identity);
        if (EditorGUI.EndChangeCheck())
        {
            Undo.RecordObject(target, "Move point");
            t.lookAtPoint = pos;
            t.Update();
        }
    }
}
```

## Custom Inspector (Serialized Field)


```cs
using System;
using UnityEngine;

enum IngredientUnit { Spoon, Cup, Bowl, Piece }

// Custom serializable class
[Serializable]
public class Ingredient
{
    public string name;
    public int amount = 1;
    public IngredientUnit unit;
}

public class Recipe : MonoBehaviour
{
    public Ingredient potionResult;
    public Ingredient[] potionIngredients;
}
```

```cs
using UnityEditor;
using UnityEngine;

// IngredientDrawer
[CustomPropertyDrawer(typeof(Ingredient))]
public class IngredientDrawer : PropertyDrawer
{
    // Draw the property inside the given rect
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
        // Using BeginProperty / EndProperty on the parent property means that
        // prefab override logic works on the entire property.
        EditorGUI.BeginProperty(position, label, property);

        // Draw label
        position = EditorGUI.PrefixLabel(position, GUIUtility.GetControlID(FocusType.Passive), label);

        // Don't make child fields be indented
        var indent = EditorGUI.indentLevel;
        EditorGUI.indentLevel = 0;

        // Calculate rects
        var amountRect = new Rect(position.x, position.y, 30, position.height);
        var unitRect = new Rect(position.x + 35, position.y, 50, position.height);
        var nameRect = new Rect(position.x + 90, position.y, position.width - 90, position.height);

        // Draw fields - passs GUIContent.none to each so they are drawn without labels
        EditorGUI.PropertyField(amountRect, property.FindPropertyRelative("amount"), GUIContent.none);
        EditorGUI.PropertyField(unitRect, property.FindPropertyRelative("unit"), GUIContent.none);
        EditorGUI.PropertyField(nameRect, property.FindPropertyRelative("name"), GUIContent.none);

        // Set indent back to what it was
        EditorGUI.indentLevel = indent;

        EditorGUI.EndProperty();
    }
}
```

## Custom Editor Window

```cs
using UnityEditor;
using UnityEngine;

public class MyWindow : EditorWindow
{
    string myString = "Hello World";
    bool groupEnabled;
    bool myBool = true;
    float myFloat = 1.23f;
    
    // Add menu item named "My Window" to the Window menu
    [MenuItem("Window/My Window")]
    public static void ShowWindow()
    {
        //Show existing window instance. If one doesn't exist, make one.
        EditorWindow.GetWindow(typeof(MyWindow));
    }
    
    void OnGUI()
    {
        GUILayout.Label ("Base Settings", EditorStyles.boldLabel);
        myString = EditorGUILayout.TextField ("Text Field", myString);
        
        groupEnabled = EditorGUILayout.BeginToggleGroup ("Optional Settings", groupEnabled);
            myBool = EditorGUILayout.Toggle ("Toggle", myBool);
            myFloat = EditorGUILayout.Slider ("Slider", myFloat, -3, 3);
        EditorGUILayout.EndToggleGroup ();
    }
}
```

## OnValidate

## Tree View

## Gizmos

```cs
using UnityEngine;
public class GizmosExample : MonoBehaviour
{
    void OnDrawGizmosSelected()
    {
        // Draw a yellow cube at the transform position
        Gizmos.color = Color.yellow;
        Gizmos.DrawWireCube(transform.position, new Vector3(10, 10, 10));
    }
}
```

## Handles

```cs
using UnityEditor;
using UnityEngine;
using System.Collections;

//this class should exist somewhere in your project
public class WireArcExample : MonoBehaviour
{
    public float shieldArea;
}

// Create a 180 degrees wire arc with a ScaleValueHandle attached to the disc
// that lets you modify the "shieldArea" value in the WireArcExample
[CustomEditor(typeof(WireArcExample))]
public class DrawWireArc : Editor
{
    void OnSceneGUI()
    {
        Handles.color = Color.red;
        WireArcExample myObj = (WireArcExample)target;
        Handles.DrawWireArc(myObj.transform.position, myObj.transform.up, -myObj.transform.right, 180, myObj.shieldArea);
        myObj.shieldArea = (float)Handles.ScaleValueHandle(myObj.shieldArea, myObj.transform.position + myObj.transform.forward * myObj.shieldArea, myObj.transform.rotation, 1, Handles.ConeHandleCap, 1);
    }
}
```

## Assets

### AssetModificationProcessor

# Unity Debug

## Logging

### Debug.Log

### Debug.LogWarning

### Debug.LogError

### Debug.LogException

### Debug.Assert

## Drawing

### Debug.DrawLine

### Debug.Drawray

## Other

### Debug.Break

# Unity OnGUI

# CLI

# Excel

# .NET MAUI
