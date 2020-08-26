[Back to Home](index.md)

# Alert Panel Manager

This was a script to handle incoming alerts from PTC Thingworx and an Azure back-end. It's only one piece of the complete system, and handled the display of Alert Boxes on a panel on the side of the application window.

``` C#
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class AlertPanelManager : MonoBehaviour
{
    [SerializeField]
    private GameObject alertPanelElementPrefab;
    [SerializeField]
    private Transform scrollViewContentPanel;

    [SerializeField]
    private float animationDuration;

    private List<GameObject> alertPanelElements = new List<GameObject>();

    private RectTransform rectTransform;

    private bool useAnimation = false;
    public bool UseAnimation
    {
        get
        {
            return useAnimation;
        }
        set
        {
            useAnimation = value;
        }
    }
                                                                   
    public void OnEnable()
    {
        if (rectTransform != null && UseAnimation)
        {
            StartCoroutine(OpenAlertPanel());
        }
        else
        {
            rectTransform = GetComponent<RectTransform>();
        }
    }

    void Start()
    {
        rectTransform = GetComponent<RectTransform>();

        if (UseAnimation)
        {
            StartCoroutine(OpenAlertPanel());
        }
    }
    public void ResetScale()
    {
        rectTransform.localScale = Vector3.one;
    }

    public void InvokeOpenAlertPanel()
    {
        StartCoroutine(OpenAlertPanel());
    }

    public void InvokeCloseAlertPanel()
    {
        StartCoroutine(CloseAlertPanel());
    }

    public IEnumerator OpenAlertPanel()
    {
        float elapsedTime = 0;
        // Stores the initial value of the scale of the alert panel on the x axis
        float width;

        // While elapsed time is less than the duration specified
        while (elapsedTime < animationDuration)
        {
            // Increase the elapsedTime
            elapsedTime += Time.deltaTime;

            // Lerp the value of the x scale to zero using an alpha
            width = Mathf.Lerp(0, 1, elapsedTime / animationDuration);

            // Set the RectTransform to use the new y scale
            rectTransform.localScale = new Vector3(width, rectTransform.localScale.y, rectTransform.localScale.z);

            yield return null;
        }
    }

    public IEnumerator CloseAlertPanel()
    {
        float elapsedTime = animationDuration;
        // Stores the initial value of the scale of the dashboard on the x axis
        float xScale = rectTransform.localScale.x;
        float width;

        // While elapsed time is less than the duration specified
        while (elapsedTime > 0)
        {
            // Increase the elapsedTime
            elapsedTime -= Time.deltaTime;

            // Lerp the value of the x scale to zero using an alpha
            width = Mathf.Lerp(0, xScale, elapsedTime / animationDuration);

            // Set the RectTransform to use the new x scale
            rectTransform.localScale = new Vector3(width, rectTransform.localScale.y, rectTransform.localScale.z);

            yield return null;
        }
        gameObject.SetActive(false);
    }

    public void RefreshAlertPanelForActiveDrillDownObject()
    {
        RefreshAlertPanel(GameManager.instance.activeDrillDownObject.GetComponent<Location>());
    }

    public void RefreshAlertPanel(Location locationIn)
    {
        Location location = new Location();
        List<ThingWorxAlert> alerts = new List<ThingWorxAlert>();

        if (!locationIn)
        {
            // Get Location script component of the Active Drill Down Object
            location = GameManager.instance.activeDrillDownObject.GetComponent<Location>();
            
            // If a location script isn't attached to the active drill down object
            if (!location)
            {
                // Print error/message to console
                Debug.Log("Currently active drill down object does not have a location script attached.");
                // return out of method
                return;
            }
        }
        else
        {
            location = locationIn;
        }


        // If the list of alert panel elements is greater than 0
        if (alertPanelElements.Count > 0)
        {
            // Destroy each gameobject in the scroll panel
            foreach (GameObject alertElement in alertPanelElements)
            {
                Destroy(alertElement);
            }

            // Clear the list
            alertPanelElements.Clear();
        }

        // If the Location Component on the Active Drill Down Object's Filter Mode is set to a particular value
        if (location.filterMode == Location.FilterMode.SITE)
        {
            // Get a list of all the alerts filtered by that value
            alerts = AlertManager.instance.FilterBySite(location.Site);
        }
        else if (location.filterMode == Location.FilterMode.BUILDING)
        {
            alerts = AlertManager.instance.FilterByBuilding(location.Building);
        }
        else if (location.filterMode == Location.FilterMode.FLOOR)
        {
            alerts = AlertManager.instance.FilterByFloor(location.Floor);
        }
        else if (location.filterMode == Location.FilterMode.ROOM)
        {
            alerts = AlertManager.instance.FilterByRoom(location.Room);
        }
        else if (location.filterMode == Location.FilterMode.DEVICE)
        {
            alerts = AlertManager.instance.FilterByDevice(location.Device);
        }

        // For each filtered alert
        foreach (ThingWorxAlert alert in alerts)
        {
            // Instantiate a UI Element in the scroll view content panel
            GameObject go = Instantiate(alertPanelElementPrefab, scrollViewContentPanel);

            // Get the AlertPanelAlertElement component on the instantiated UI element and set the text
            AlertPanelAlertElement alertElement = go.GetComponent<AlertPanelAlertElement>();

            DateTime newDate = DateTime.Parse("1 Jan 1970").AddMilliseconds(double.Parse(alert.OccuredAt));

            alertElement.TimeText.text = newDate.ToString("HH:mm");
            alertElement.DateText.text = newDate.ToString("dd/MM/yyyy");
            alertElement.DescriptionText.text = alert.Description;
            alertElement.DeviceName = alert.source;

            // Add the instantiated UI element to the AlertPanelElements array
            alertPanelElements.Add(go);
        }
    }
}
```

[Back to Home](index.md)
