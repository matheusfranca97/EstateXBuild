
# Unity WebGL Browser Interaction Example

This repository demonstrates how to interact between a Unity WebGL build and the browser using JavaScript.

## Communication Reference

Refer to the Unity documentation for interacting between the browser and Unity: [Web Interaction](https://docs.unity3d.com/Manual/web-interacting-browser-js-to-unity.html).

### 1. Create a JSLib File

Ive Created a JSLib file with the following code. This file allows Unity to interact with browser JavaScript.

```javascript
mergeInto(LibraryManager.library, {
    SendDataToParent: function(data) {
        var jsonString = UTF8ToString(data);
        console.log("JSON string received:", jsonString);

        // Verifica se a string pode ser corretamente parseada em um objeto
        try {
            var jsonData = JSON.parse(jsonString);
            console.log("Parsed JSON data:", jsonData);
        } catch (e) {
            console.error("Failed to parse JSON:", e);
        }

        window.parent.postMessage(jsonString, "*");
    }
});
```

This code defines a JavaScript function `SendDataToParent` that opens a new browser tab with the provided URL.

### 2. Call the JavaScript Function from C#

In your Unity C# script, you can now call the `SendDataToParent` function defined in your JSLib file.

```csharp
[System.Serializable]
public class FinalScoreData
{
    public int score;
    public string message;
}

public class BackendCommunicationManager : MonoBehaviour
{
    public static BackendCommunicationManager Instance { get; private set; }

#if UNITY_WEBGL && !UNITY_EDITOR
    [DllImport("__Internal")]
    private static extern void SendDataToParent(string data);
#endif

    private void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    public void SendFinalScoreToParent(int finalPlayerScore)
    {
        FinalScoreData data = new FinalScoreData
        {
            score = finalPlayerScore,
            message = "Hello, I'm Matheus sending you a message from EstateX game o/"
        };

        string jsonData = JsonUtility.ToJson(data);
        Debug.Log("JSON data being sent: " + jsonData);

#if UNITY_WEBGL && !UNITY_EDITOR
    SendDataToParent(jsonData);
#else
        Debug.Log("SendDataToParent is only available in WebGL builds.");
#endif
    }
}
```

