# Burn Rate Alert
It is a Azure burn rate calculation on logic app.

Everyday at UTC 6am, it grabs the daily cost of all resources used on the day before yesterday (TDBY) and also three days before (ThreeDayBefore) by using Consumption API. Afterwards, it sums up each day's cost. The burn rate is calculated as an increase rate:

Burn Rate = (TDBY - ThreeDayBefore)/ThreeDayBefore

The reason why not calculate difference between yesterday and today is because some costs are calculated once per two days. TDBY and ThreeDayBefore turn out to be more stable. 

## Step 1: Create an logic app

Follow **only** steps in the section "Create your logic app" here <https://docs.microsoft.com/en-us/azure/logic-apps/quickstart-create-first-logic-app-workflow>.

Now you will have an empty logic app. Open it and click "blank logic app" in the templates.  

Pick "code view", delete the content and copy paste the whole logicapptemplate.txt here <https://github.com/wcex1994/burnrate/blob/master/logicapptemplate> into the window. Now go back to the designer view.

## Step 2: Subsciption list

In the "Sublist" variable, add in a list of subscription ID that you would like to calculate the burn rate.

![alt text](https://github.com/wcex1994/burnrate/blob/master/pictures/subscriptionid.jpg "sub_id")

You can grab related information with the following PowerShell commands locally:

```POWERSHELL
Login-AzureRmAccount

Get-AzureRmSubscription | Export-Csv "C:\SOME_PATH\Azure Subscription.csv"
```

## Step 3: Managed Identity and IAM

In order to be authenticated and aurthorized to call the Azure Comsumption API, we use Managed Identity and Identity and Access Management (IAM). Notice there might be better methods depends on your scenario.

1. Turn on the Managed Identity in the logic app:

    Go to your logic app --> identity --> click "On"

    ![alt text](https://github.com/wcex1994/burnrate/blob/master/pictures/turn_on_mi.jpg "turn_on_MI")

2. Grant your logic app "Contributor" role in the subscriptions that you would like to calculate the burn rate.

    Subscription --> IAM --> Add Role
    ![alt text](https://github.com/wcex1994/burnrate/blob/master/pictures/iam_s1.jpg "turn_on_MI")

    Choose "Contributor" and search for your logic app name --> "Save"

    ![alt text](https://github.com/wcex1994/burnrate/blob/master/pictures/iam_s2.jpg "turn_on_MI")

    If you have many subscriptons, you can also grant authorization by PowerShell, see <https://docs.microsoft.com/en-us/azure/role-based-access-control/tutorial-role-assignments-user-powershell#grant-access>.


## Step 4: Add Action

In condition 3, edit threshold and actions once your burn rate is over the threshold. For example, you can send an email in outlook.

![alt text](https://github.com/wcex1994/burnrate/blob/master/pictures/actions.jpg "action")

## Further Adjustment

1. Error control:
    For example, if the previous day is 0, then how does it work to calculate the burn rate, you cannot divide something by 0.

2. First day of month:

    The Consumption.UsageDetails.List API only returns current billing period's usage. Azure counts one month as a billing period. If you are on Feb 1st, then to calculate Jan 30th vs. Jan 31st burn rate, you would need to call Consumption.UsageDetails.ListByBillingPeriod API with '201901' as the billing period. See here <https://docs.microsoft.com/en-us/rest/api/consumption/usagedetails/listbybillingperiod> for more details.

## Additional Information

* Consumption.UsageDetails.List API: <https://docs.microsoft.com/en-us/rest/api/consumption/usagedetails/list>
* Consumption.Budget API: <https://docs.microsoft.com/en-us/rest/api/consumption/budgets>
* Managed Identity: <https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview>
* IAM: <https://azure.microsoft.com/en-ca/product-categories/identity/>

