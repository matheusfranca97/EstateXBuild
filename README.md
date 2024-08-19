
# Unity WebGL Browser Interaction Example

This repository demonstrates how to interact between a Unity WebGL build and the browser using JavaScript.

## Communication Reference

Refer to the Unity documentation for detailed guidance on interacting between the browser and Unity: [Unity WebGL Browser Interaction](https://docs.unity3d.com/Manual/web-interacting-browser-unity-to-js.html).

### 1. Create a JSLib File

I've created a JSLib file with the following code. This file allows Unity to interact with browser JavaScript.

```javascript
mergeInto(LibraryManager.library, {
    SendDataToParent: function(data) {
        var jsonString = UTF8ToString(data);
        console.log("JSON string received:", jsonString);

        // Parse the JSON string and log the parsed object
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

This code defines a JavaScript function `SendDataToParent` that receives data from Unity, processes it, and sends it to the parent window.

### 2. Call the JavaScript Function from C#

In my Unity C# script, I can now call the `SendDataToParent` function defined in my JSLib file. The script also provides an example of how to expose a function that JavaScript can call to send data back to Unity.

```csharp
[System.Serializable]
public class GameDataDTO
{
    public int points;
    public int playCoins;
    public int timeCoins;
    public int boostCoins;
}

public class BackendCommunicationManager : MonoBehaviour
{
    public static BackendCommunicationManager Instance { get; private set; }
    public GameDataSO CurrentGameData;

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

    public void SendFinalScoreToParent()
    {
        GameDataDTO data = new GameDataDTO
        {
            points = CurrentGameData.points,
            playCoins = CurrentGameData.playCoins,
            timeCoins = CurrentGameData.timeCoins,
            boostCoins = CurrentGameData.boostCoins
        };

        string jsonData = JsonUtility.ToJson(data);
        Debug.Log("JSON data being sent: " + jsonData);

#if UNITY_WEBGL && !UNITY_EDITOR
    SendDataToParent(jsonData);
#else
        Debug.Log("SendDataToParent is only available in WebGL builds.");
#endif
    }


    public void ReceivePlayerData(string json)
    {
        ResetGameData();
        Debug.Log("This is the data from NextJs: " + json);
        GameDataDTO receivedData = JsonUtility.FromJson<GameDataDTO>(json);
        CurrentGameData.playCoins = receivedData.playCoins;
        CurrentGameData.timeCoins = receivedData.timeCoins;
        CurrentGameData.boostCoins = receivedData.boostCoins;
        CurrentGameData.points = receivedData.points;
    }
}
```

### 3. Sending Data from JavaScript to Unity

To send data to Unity, you can use the `SendMessage` function, which is designed to communicate with the Unity WebGL instance. Here’s how you can do it:

1. **Access the Unity instance**: Typically, when Unity WebGL is initialized, it creates an instance (`unityInstance`) that you can use to interact with the Unity game.

2. **Send a JSON message**: Use the `SendMessage` function to send a JSON message to the Unity game. For example, if you want to send a JSON with player data, you would do the following:

```javascript
// Example of sending data to Unity from JavaScript
unityInstance.SendMessage('BackendCommunicationManager', 'ReceivePlayerData', '{"score": 100, "message": "JSON example"}');
```

- **`unityInstance`**: This refers to the Unity WebGL instance.
- **`BackendCommunicationManager`**: The name of the GameObject in Unity where the `BackendCommunicationManager` script is attached (use the same name).
- **`ReceivePlayerData`**: The name of the method in Unity that will process the received data (use the same name).
- **`'{"score": 100, "message": "Hello from Next.js"}'`**: The JSON string you want to send to Unity.

### Summary

- **Function Exposure**: The `ReceivePlayerData` function in the Unity script is exposed to allow JavaScript to send data back to Unity. 
- **Sending Data**: Use the `SendMessage` function in JavaScript to interact with Unity by calling the exposed method and passing the required data.
