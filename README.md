```
using UnityEngine;
using UnityEditor;

public class MeshCombinerEditor : EditorWindow
{
    [MenuItem("Tools/Combine Meshes")]
    public static void ShowWindow()
    {
        GetWindow<MeshCombinerEditor>("Combine Meshes");
    }

    private void OnGUI()
    {
        if (GUILayout.Button("Combine Selected Meshes"))
        {
            CombineSelectedMeshes();
        }
    }

    private void CombineSelectedMeshes()
    {
        GameObject[] selectedObjects = Selection.gameObjects;
        if (selectedObjects.Length == 0) return;

        CombineInstance[] combine = new CombineInstance[selectedObjects.Length];
        for (int i = 0; i < selectedObjects.Length; i++)
        {
            MeshFilter mf = selectedObjects[i].GetComponent<MeshFilter>();
            if (mf == null) continue;

            combine[i].mesh = mf.sharedMesh;
            combine[i].transform = mf.transform.localToWorldMatrix;
        }

        GameObject combinedObject = new GameObject("CombinedMesh");
        MeshFilter combinedMF = combinedObject.AddComponent<MeshFilter>();
        combinedMF.mesh = new Mesh();
        combinedMF.mesh.CombineMeshes(combine);

        MeshRenderer combinedMR = combinedObject.AddComponent<MeshRenderer>();
        combinedMR.sharedMaterial = selectedObjects[0].GetComponent<MeshRenderer>()?.sharedMaterial;
    }
}
```
