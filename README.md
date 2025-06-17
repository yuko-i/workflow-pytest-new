```
// Assets/Editor/ExportChildrenToGLB.cs
#if UNITY_EDITOR
using UnityEditor;
using UnityEngine;
using System.IO;
using System.Linq;

#if SICCITY_GLTFUTILITY
using Siccity.GLTFUtility;
#elif GLTFAST_EXPORT
using GLTFast.Export;
using GLTFast;
#endif

public static class ExportChildrenToGLB
{
    //--------------------------------------------------------------------
    // メニュー項目
    //--------------------------------------------------------------------
    [MenuItem("Tools/Export/Children to GLB (Downloads)")]
    private static void ExportChildren()
    {
        var parent = Selection.activeGameObject;
        if (parent == null)
        {
            EditorUtility.DisplayDialog(
                "Export Children to GLB",
                "Hierarchy で親オブジェクトを 1 つ選択してください。",
                "OK");
            return;
        }

        // ダウンロードフォルダの取得
        var downloadsPath = GetDownloadsPath();
        Directory.CreateDirectory(downloadsPath);

        // 子オブジェクトを列挙
        var children = parent.transform.Cast<Transform>().ToArray();
        if (children.Length == 0)
        {
            EditorUtility.DisplayDialog(
                "Export Children to GLB",
                "選択したオブジェクトに子がありません。",
                "OK");
            return;
        }

        foreach (var child in children)
        {
            var go = child.gameObject;
            var safeName = MakeSafeFileName(go.name) + ".glb";
            var fullPath = Path.Combine(downloadsPath, safeName);

            // 既に存在したら上書き
            if (File.Exists(fullPath)) File.Delete(fullPath);

#if SICCITY_GLTFUTILITY // -------- GLTFUtility --------
            Exporter.ExportGLB(go, fullPath);

#elif GLTFAST_EXPORT    // -------- glTFast --------
            var settings = new ExportSettings
            {
                Format = GltfFormat.Binary,               // GLB
                FileConflictResolution = FileConflictResolution.Overwrite
            };
            var exporter = new GameObjectExport(settings);
            exporter.AddScene(go.transform);
            exporter.Save(fullPath);
            exporter.Dispose();

#else                   // -------- com.unity.formats.gltf (公式) --------
            using var stream = new FileStream(fullPath, FileMode.Create, FileAccess.Write);
            var exporter = new UnityEditor.Formats.Gltf.Exporters.GLTFSceneExporter(
                new Transform[] { go.transform });
            exporter.SaveGLB(stream);
#endif

            Debug.Log($"Exported: {fullPath}");
        }

        // 書き込んだフォルダを開く
#if UNITY_EDITOR_OSX
        System.Diagnostics.Process.Start("open", downloadsPath);
#elif UNITY_EDITOR_WIN
        System.Diagnostics.Process.Start("explorer.exe", downloadsPath.Replace("/", "\\"));
#endif
    }

    //--------------------------------------------------------------------
    // ダウンロードフォルダ取得（Windows / macOS 共通）
    //--------------------------------------------------------------------
    private static string GetDownloadsPath()
    {
        var home = System.Environment.GetFolderPath(System.Environment.SpecialFolder.UserProfile);
        return Path.Combine(home, "Downloads");
    }

    //--------------------------------------------------------------------
    // ファイル名に使えない文字を置換
    //--------------------------------------------------------------------
    private static string MakeSafeFileName(string name)
    {
        foreach (var c in Path.GetInvalidFileNameChars())
            name = name.Replace(c, '_');
        return name;
    }
}
#endif
```
