[Back to Home](../index.md)

# Exploded View

I created this after seeing the Unreal Engine Product Viewer Template - we needed something similar but for Unity. It’s fairly basic, and the system can be improved, but it got the job done at the time.

These scripts allow an object with multiple children to be "exploded" into various positions - allowing for easier inspection of the individual meshes. This is used in Digital Twins and Product Viewers, for example.

### Custom Editor:

``` C#
using UnityEngine;
using UnityEditor;

[CustomEditor(typeof(DrillDownComponent_ExplodedView))]
public class ExplodedViewEditor : Editor
{
    public override void OnInspectorGUI()
    {
        base.OnInspectorGUI();

        DrillDownComponent_ExplodedView explodedView = (DrillDownComponent_ExplodedView) target;

        GUILayout.BeginHorizontal();

        if (GUILayout.Button("Fill Array with Children"))
        {
            foreach (var child in explodedView.transform.GetComponentsInChildren<MeshRenderer>())
            {
                explodedView.objectsToExplode.Add(child.transform);
            }
        }

        if (GUILayout.Button("Clear Array"))
        {
            explodedView.objectsToExplode.Clear();
        }

        GUILayout.EndHorizontal();

        GUILayout.BeginHorizontal();

        if (GUILayout.Button("Set Initial Transform"))
        {
            explodedView.initialPositions = new Vector3[explodedView.objectsToExplode.Count];
            explodedView.initialRotations = new Quaternion[explodedView.objectsToExplode.Count];

            for (int i = 0; i < explodedView.objectsToExplode.Count; i++)
            {
                explodedView.initialPositions[i] = explodedView.objectsToExplode[i].position;
                explodedView.initialRotations[i] = explodedView.objectsToExplode[i].rotation;
            }
        }

        if (GUILayout.Button("Set Exploded Transform"))
        {
            explodedView.finalPositions = new Vector3[explodedView.objectsToExplode.Count];
            explodedView.finalRotations = new Quaternion[explodedView.objectsToExplode.Count];

            for (int i = 0; i < explodedView.objectsToExplode.Count; i++)
            {
                explodedView.finalPositions[i] = explodedView.objectsToExplode[i].position;
                explodedView.finalRotations[i] = explodedView.objectsToExplode[i].rotation;
            }
        }

        GUILayout.EndHorizontal();

        if (GUILayout.Button("Reset Transforms"))
        {   
            for (int i = 0; i < explodedView.objectsToExplode.Count; i++)
            {
                explodedView.objectsToExplode[i].position = explodedView.initialPositions[i];
                explodedView.objectsToExplode[i].rotation = explodedView.initialRotations[i];
            }
        }
    }
}
```

### Implementation:


``` C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

//[RequireComponent(typeof(DrillDownComponent_Select))]
public class DrillDownComponent_ExplodedView : MonoBehaviour
{
    public List<Transform> objectsToExplode;
    public float duration;

    [HideInInspector]
    public float lerp = 0;
    //[HideInInspector]
    public bool exploded;
    //[HideInInspector]
    public bool explode;
    //[HideInInspector]
    public bool retract;
    [HideInInspector]
    public Vector3[] initialPositions;
    [HideInInspector]
    public Quaternion[] initialRotations;
    [HideInInspector]
    public Vector3[] finalPositions;
    [HideInInspector]
    public Quaternion[] finalRotations;

    // Start is called before the first frame update
    void Start()
    {
        exploded = false;

        for (int i = 0; i < objectsToExplode.Count; i++)
        {
            objectsToExplode[i].position = initialPositions[i];
            objectsToExplode[i].rotation = initialRotations[i];
        }
    }

    // Update is called once per frame
    void Update()
    {
        if (explode)
        {
            if (!exploded && lerp <= duration)
            {
                lerp += Time.deltaTime / duration;

                for (int i = 0; i < objectsToExplode.Count; i++)
                {
                    objectsToExplode[i].position = Vector3.Lerp(initialPositions[i], finalPositions[i], lerp);
                }

                for (int i = 0; i < objectsToExplode.Count; i++)
                {
                    objectsToExplode[i].rotation = Quaternion.Lerp(initialRotations[i], finalRotations[i], lerp);
                }
            }
            else
            {
                exploded = true;
                explode = false;
                lerp = 0;
            }
        }

        if (retract)
        {
            if (exploded && lerp <= duration)
            {
                lerp += Time.deltaTime / duration;

                for (int i = 0; i < objectsToExplode.Count; i++)
                {
                    objectsToExplode[i].position = Vector3.Lerp(finalPositions[i], initialPositions[i], lerp);
                }

                for (int i = 0; i < objectsToExplode.Count; i++)
                {
                    objectsToExplode[i].rotation = Quaternion.Lerp(finalRotations[i], initialRotations[i], lerp);
                }
            }
            else
            {
                exploded = false;
                retract = false;
                lerp = 0;
            }
        }
    }
}
```

[Back to Home](../index.md)
