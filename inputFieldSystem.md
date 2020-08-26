[Back to Home](index.md)

# Input Field System

This script ensures that the user types in the correct information as per the type designated to the InputField. It’s not complex, but it’s definitely a necessity for log in and sign up pages within applications. This was done for a mobile app with Augmented Reailty functionality, among other things.

### Input Field Validation:

``` C#
using System.Collections;
using System.Collections.Generic;
using TMPro;
using UnityEngine;

[RequireComponent(typeof(TMP_InputField))]
public class InputFieldValidation : MonoBehaviour
{
    private TMP_InputField inputField;

    public enum InputType { Standard, Email, Password };
    [SerializeField]
    private InputType inputType;
    
    [SerializeField]
    private PasswordStrength minimumPasswordStrength;

    [SerializeField]
    private TMP_Text errorText;

    [SerializeField]
    private bool requiredField;

    [SerializeField]
    private GameObject showPasswordButton;

    private bool isValid;
    public bool IsValid
    {
        get
        {
            ValidateInput();
            return isValid;
        }
        private set
        {
            isValid = value;
        }
    }

    #region EDITOR

    /// <summary>
    /// Updates the content type component on the TMP_InputField so the content type doesn't have to
    /// be set here and on that script manually.
    /// </summary>
    private void OnValidate()
    {
        inputField = GetComponent<TMP_InputField>();

        switch (inputType)
        {
            case InputType.Standard:
                inputField.contentType = TMP_InputField.ContentType.Standard;
                break;
            case InputType.Email:
                inputField.contentType = TMP_InputField.ContentType.EmailAddress;
                break;
            case InputType.Password:
                inputField.contentType = TMP_InputField.ContentType.Password;
                break;
        }
    }

    #endregion

    private void Start()
    {
        inputField = GetComponent<TMP_InputField>();
        inputField.onEndEdit.AddListener(OnEndEdit);

        if (showPasswordButton == null)
        {
            Debug.LogError($"Show Password Button reference not assigned for Input Field: {inputField.gameObject.name}.");
        }
        else
        {
            if (inputType == InputType.Password)
            {
                showPasswordButton.SetActive(true);
            }
            else
            {
                showPasswordButton.SetActive(false);
            }
        }

        if (errorText == null)
        {
            Debug.LogError($"Error text reference not assigned for Input Field: {inputField.gameObject.name}.");
        }
        else
        {
            errorText.text = "";
        }

        IsValid = false;
    }

    private void OnEndEdit(string arg0)
    {
        ValidateInput();
    }

    private void ValidateInput()
    {
        if (!IsInputFieldRequiredAndEmpty())
        {
            switch (inputType)
            {
                case InputType.Standard:
                    // No validation required
                    break;
                case InputType.Email:
                    EmailValidation();
                    break;
                case InputType.Password:
                    PasswordValidation();
                    break;
            }
        }
    }

    private void EmailValidation()
    {
        if (EmailCheck.IsValidEmail(inputField.text))
        {
            errorText.text = "";
            IsValid = true;
        }
        else
        {
            errorText.text = "* Please enter a valid email address";
            IsValid = false;
        }
    }

    private void PasswordValidation()
    {
        PasswordStrength passwordStrength = PasswordCheck.GetPasswordStrength(inputField.text);

        if (passwordStrength >= minimumPasswordStrength)
        {
            errorText.text = "";
            IsValid = true;
        }
        else
        {
            errorText.text = "* Password does not meet minimum strength requirements.";
            IsValid = false;
        }
    }

    private bool IsInputFieldRequiredAndEmpty()
    {
        if (requiredField && string.IsNullOrEmpty(inputField.text))
        {
            errorText.text = "* Field cannot be empty.";
            IsValid = false;
            return true;
        }
        else
        {
            errorText.text = "";
            IsValid = true;
            return false;
        }
    }

    public void OnClick_ShowPassword()
    {
        inputField.contentType = TMP_InputField.ContentType.Standard;
        inputField.ForceLabelUpdate();
    }

    public void OnClick_HidePassword()
    {
        inputField.contentType = TMP_InputField.ContentType.Password;
        inputField.ForceLabelUpdate();
    }
}
```

### 

[Back to Home](index.md)
