# Login Page

A simple login page.

``` C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using TMPro;

public class LogInPage : MonoBehaviour
{
    [SerializeField]
    private TMP_Text errorText;
    [SerializeField]
    private List<InputFieldValidation> inputFieldValidations = new List<InputFieldValidation>();

    private bool canLogin;

    private void OnEnable()
    {
        EventManager.CloseAllPages += SetPanelInactive;
        EventManager.OpenLoginPage += SetPanelActive;
        EventManager.CloseLoginPage += SetPanelInactive;

        EventManager.LoginSuccess += OnLoginSuccessful;
        EventManager.LoginSuccessForceChangePassword += OnLoginSuccessfulForceChangePassword;
        EventManager.LoginFailed += OnLoginUnsuccessful;
    }

    private void OnDisable()
    {
        EventManager.CloseAllPages -= SetPanelInactive;
        EventManager.OpenLoginPage -= SetPanelActive;
        EventManager.CloseLoginPage -= SetPanelInactive;

        EventManager.LoginSuccess -= OnLoginSuccessful;
        EventManager.LoginSuccessForceChangePassword -= OnLoginSuccessfulForceChangePassword;
        EventManager.LoginFailed -= OnLoginUnsuccessful;
    }

    private void SetPanelActive()
    {
        transform.GetChild(0).gameObject.SetActive(true);
    }

    private void SetPanelInactive()
    {
        transform.GetChild(0).gameObject.SetActive(false);
    }

    void Start()
    {
        if (errorText == null)
        {
            Debug.LogError($"Error text reference not assigned for Log In Page.");
        }
        else
        {
            errorText.text = "";
        }

        if (inputFieldValidations.Count < 1)
        {
            Debug.LogError("No InputFieldValidation references assigned in the LogIn inspector.");
        }
    }

    public void OnClick_LogIn()
    {
        if (CanLogin())
        {
            Debug.Log("Login!");
            NetworkManager.Instance.Login
            (
                emailAddress: inputFieldValidations[0].gameObject.GetComponent<TMP_InputField>().text,
                password: inputFieldValidations[1].gameObject.GetComponent<TMP_InputField>().text
            );

            EventManager.OpenLoadingScreen?.Invoke();
        }
        else
        {
            Debug.Log("Stay here!");
        }
    }

    public void OnClick_Back()
    {
        EventManager.CloseAllPages?.Invoke();
        EventManager.OpenLandingPage?.Invoke();
    }

    public void OnClick_ForgotPassword()
    {
        EventManager.CloseAllPages?.Invoke();
        EventManager.OpenForgotPasswordPage?.Invoke();
    }

    private void OnLoginSuccessful()
    {
        EventManager.CloseLoadingScreen?.Invoke();
        EventManager.OpenAppbar?.Invoke();
        EventManager.OpenNavbar?.Invoke();
        EventManager.OpenNewsFeed?.Invoke();
        EventManager.CloseLoginPage?.Invoke();
    }

    private void OnLoginSuccessfulForceChangePassword()
    {
        EventManager.CloseLoadingScreen?.Invoke();
        EventManager.OpenChangePasswordPage?.Invoke();
        EventManager.CloseLoginPage?.Invoke();
    }

    private void OnLoginUnsuccessful()
    {
        errorText.text = "* Unable to log in. Please check your credentials or internet connection and try again.";
        EventManager.CloseLoadingScreen?.Invoke();
    }

    private bool CanLogin()
    {
        canLogin = true;

        foreach (InputFieldValidation inputFieldValidation in inputFieldValidations)
        {
            if (!inputFieldValidation.IsValid)
            {
                canLogin = false;
            }
        }

        return canLogin;
    }
}
```
