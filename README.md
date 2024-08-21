
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
    },

    SendExitEventToParent: function() {
        console.log("Sending exit event to parent.");

        // Create and dispatch a custom event for the exit action
        var exitEvent = new CustomEvent('unityExitEvent', { detail: { exit: true } });
        window.dispatchEvent(exitEvent);
    }
});
```

This code defines two JavaScript functions:

1. **`SendDataToParent`**: Receives data from Unity, processes it, and sends it to the parent window.
2. **`SendExitEventToParent`**: Dispatches a custom event (`unityExitEvent`) to notify the browser when the exit action is triggered in Unity.

### 2. Call the JavaScript Functions from C#

In my Unity C# script, I can now call the `SendDataToParent` and `SendExitEventToParent` functions defined in my JSLib file. The script also provides an example of how to expose a function that JavaScript can call to send data back to Unity.

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

    [DllImport("__Internal")]
    private static extern void SendExitEventToParent();
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

    public void SendExitEvent()
    {
#if UNITY_WEBGL && !UNITY_EDITOR
        SendExitEventToParent();
#else
        Debug.Log("SendExitEventToParent is only available in WebGL builds.");
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

### 3. Handling Exit Events in the Browser

To handle exit events in the browser, you can add a listener for the `unityExitEvent` custom event. This event can be used to trigger actions such as redirecting the user or reloading the page.

```javascript
// Example of handling the unityExitEvent in JavaScript
window.addEventListener('unityExitEvent', function(event) {
    console.log("Exit event received from Unity:", event.detail);
    if (event.detail.exit) {
        location.reload(); // Example action: reload the page
    }
});
```

This listener:

- **`unityExitEvent`**: Listens for the custom `unityExitEvent` dispatched by Unity when the user triggers an exit action.
- **`event.detail.exit`**: Checks if the event contains an `exit: true` value and performs an action based on that (e.g., reloading the page).

### 4. Sending Data from JavaScript to Unity

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
- **Exit Event Handling**: The `SendExitEventToParent` function and the corresponding event listener in the browser provide a way to handle exit actions triggered from Unity, allowing for custom behavior like page reloads or redirects.
