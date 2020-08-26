[Back to Home](../index.md)

# Text To Speech Handler

This was a project done with AWS Lex, and at the time getting the speech response from Lex into Unity was a bit of a challenge, so this was used to convert the response string into an audio file instead.

``` C#
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(AudioSource))]
public class TextToSpeechHandler : MonoBehaviour
{
    public static TextToSpeechHandler Instance;

    private AudioSource audioSource;

    private void Awake()
    {
        if (Instance == null)
        {
            Instance = this;
        }
        else
        {
            Destroy(gameObject);
        }
    }

    private void Start()
    {
        audioSource = GetComponent<AudioSource>();
    }

    public void CreateAudioClip(string audioString)
    {
        StartCoroutine(ClipGenerator(audioString));
    }

    IEnumerator ClipGenerator(string audioString)
    {
        byte[] rawData = Convert.FromBase64String(audioString);
        string tempFile = Application.persistentDataPath + "/bytes.ogg";
        System.IO.File.WriteAllBytes(tempFile, rawData);

        WWW loader = new WWW("file://" + tempFile);

        yield return loader;

        if (!String.IsNullOrEmpty(loader.error))
        {
            Debug.LogError(loader.error);
        }

        AudioClip audioClip = loader.GetAudioClip(false, false, AudioType.OGGVORBIS);

        float audioClipLength = audioClip.length;

        audioSource.PlayOneShot(audioClip);

        StartCoroutine(WaitForSpeechToEnd(audioClipLength));
    }

    IEnumerator WaitForSpeechToEnd(float timeToWait)
    {
        GameManager.HostTalkingBegin?.Invoke();

        yield return new WaitForSeconds(timeToWait);

        GameManager.HostTalkingComplete?.Invoke();

        if (GameManager.Instance.moveToNextState)
        {
            GameManager.Instance.moveToNextState = false;

            GameManager.Instance.gameState++;

            // If delegate != null
            GameManager.IncrementGameState?.Invoke();
        }
    }
}
```

[Back to Home](../index.md)
