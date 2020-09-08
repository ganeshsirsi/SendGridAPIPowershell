
# SendGridAPIPowershell

# How to send an email with an attachment using Powershell and SendGrid API

**In this article let us discuss Sending email with attachment using Powershell and Send Grid. This includes below steps**
**Pre-Requisite:** You need to have SendGrid Account (If you don't have one create a free Account)
**Step 1:** Register Sender in Send Grid
**Step 2:** Generate Send Grid API Key
**Step 3:** Create a Powershell function to send an email with attachment using SendGrid API
**Step 4:** Call the function using Powershell

## Register Sender in Send Grid

**Login to SendGrid account with your username and password** (Note: If you do not have a SendGrid account create one using SendGrid SignUp)
**Navigate to Settings > Sender Authentication**
**Click on Verify Single Sender**


**In a Create Sender Form, Enter all the mandatory details and Click on Create**
***Why do we need to add the Sender?***
Adding and verification of the sender is a must in SendGrid for security reasons.
Without adding the sender, you will not be able to send the email to any person.
The added sender email is used as From address in SendGrid API post request.
Without adding the sender and verifying the sender, SendGrid will not allow you to send any emails.
**Important! Once you add the Sender, You will get an email to verify the Sender, you need to click on the Verify Sender link to complete the process of adding the sender.**

## Generating API Key in SendGrid

API Key is very important and mandatory if you are using SendGrid API to Send Emails
**In order to Generate API Key, 
1.Go to Settings > Click on API Keys
2. Now, Click on Create API Key
3. Enter Your API Key Name, Choose Access and Click on Create and View
4. Once everything is done, You will see API Key
*Important!: Copy the API Key and save Somewhere, If you lose this API key you can't get it back.***
*Now we have successfully added sender and generated API key.*

Next, We need to Create a Powershell script to Call the API and Send an Email.

# Powershell Script to Sending email with Attachment using SendGrid

***In order to Send SendGrid Email We need To Address, From Address, Subject, Body, APIToken, FileName, and FilePath and AttachementType .***

## Create a function with parameters

    Function SendGridMailWithAttachment {
        param (
            [cmdletbinding()]
            [parameter()]
            [string]$ToAddress,
            [parameter()]
            [string]$FromAddress,
            [parameter()]
            [string]$Subject,
            [parameter()]
            [string]$Body,
            [parameter()]
            [string]$APIKey,
            [parameter()]
            [string]$FileName,
            [parameter()]
            [string]$FilePathWithName,
            [parameter()]
            [string]$AttachementType
       )

`$ToAddress` :This can be any address basically it is a recipient.
`$FromAddress` :This must be verified address (We have added in Step 2).
`$Subject` :Any string this will be shown as the subject line in the inbox
`$Body ` :Body text if you want to write anything example "Please find attachment"
`$APIKey` :This is your API Key which is generated in the above step.
`$FileName` : Important! This can be any string. Specify file with extension, not File path
Note: This filename will be shown when the user receives the email.
Example: I am attaching 1.html but $FileName the value I am giving is SampleFile.html. So in this case even though you are attaching 1.html for user it will be shown as SampleFile.html
Example : $FileName: myAttachement.html (Do not mention any path)

`$FilePathWithName`: This is the original File we want to add it as an attachment. This file will be converted to base64 attachment
Example: D://1.html

`$AttachementType` :This is the MIME type for attachment Refer: Mime Attachment Types.
Example: text/html (for Text or HTML file as an attachement)

## Create a function body

Send Grid won't allow you to send plain files as an attachment through SendGrid API. So In order to send any file as an attachment you need to create that into base64 format First.
Let us consider, We are sending the 1.html file as an attachment to SendGrid.
Using PowerShell we can covert .html file into base64 format.
`$FileContent` = get-content $FilePathWithName
`$ConvertToBytes` = [System.Text.Encoding]::UTF8.GetBytes($FileContent)
`$EncodedFile` = [System.Convert]::ToBase64String($ConvertToBytes)
In the above, the file is converted into base64 format and stored in the variable $EncodedFile.

## Create SendGrid JSON Body

We need to create a send grid body object. Which will have information such as, to address, from address, body text, attachment, etc.
# Body with attachement for SendGrid
    $SendGridBody = @{
        "personalizations" = @(
            @{
              "to"= @(
                      @{
                          "email" = $ToAddress
                       }
                 )
              "subject" = $Subject
            }
        )
               "content"= @(
                             @{
                                "type" = "text/html"
                                 "value" = $Body
                               }
                 )
                "from"  = @{
                            "email" = $FromAddress
                           }
                "attachments" = @(
                                    @{
                                        "content"=$EncodedFile
                                        "filename"=$FileName
                                        "type"= $AttachementType
                                        "disposition"="attachment"
                                     }
                 )
     }

$BodyJson = $SendGridBody | ConvertTo-Json -Depth 4

## Create header for Sendgrid Email API

We are going to pass the API key in the header.
#Header for SendGrid API

    $Header = @{
              "authorization" = "Bearer $APIKey"
    }

## Invoke SendGrid API in your function

Send the email through SendGrid API
    $Parameters = @{
        Method      = "POST"
        Uri         = "https://api.sendgrid.com/v3/mail/send"
        Headers     = $Header
        ContentType = "application/json"
        Body        = $BodyJson
    }
    Invoke-RestMethod @Parameters

## Putting Everything Together. The Final Powershell Script

    Function SendGridMailWithAttachment {
    param (
        [cmdletbinding()]
        [parameter()]
        [string]$ToAddress,
        [parameter()]
        [string]$FromAddress,
        [parameter()]
        [string]$Subject,
        [parameter()]
        [string]$Body,
        [parameter()]
        [string]$APIKey,
        [parameter()]
        [string]$FileName,
        [parameter()]
        [string]$FileNameWithFilePath,
		[parameter()]
        [string]$AttachementType
    )

    #Convert File to Base64
    $FileContent = get-content $FileNameWithFilePath
    $ConvertToBytes = [System.Text.Encoding]::UTF8.GetBytes($FileContent)
    $EncodedFile = [System.Convert]::ToBase64String($ConvertToBytes)

    # Body with attachement for SendGrid
    $SendGridBody = @{
        "personalizations" = @(
            @{
                "to"= @(
                              @{
                                   "email" = $ToAddress
                               }
                 )

                "subject" = $Subject
            }
        )

                "content"= @(
                              @{
                                    "type" = "text/html"
                                    "value" = $Body
                               }
                 )

                "from"  = @{
                            "email" = $FromAddress
                           }

                "attachments" = @(
                                    @{
                                        "content"=$EncodedFile
                                        "filename"=$FileName
                                        "type"= $AttachementType
                                        "disposition"="attachment"
                                     }
               )
         }

    $BodyJson = $SendGridBody | ConvertTo-Json -Depth 4


    #Header for SendGrid API
    $Header = @{
        "authorization" = "Bearer $APIKey"
    }

    #Send the email through SendGrid API
    $Parameters = @{
        Method      = "POST"
        Uri         = "https://api.sendgrid.com/v3/mail/send"
        Headers     = $Header
        ContentType = "application/json"
        Body        = $BodyJson
    }
    Invoke-RestMethod @Parameters
    }
    

Invoke Above method with below parameters 

$Parameters = @{
    ToAddress   = "example@gmail.com"
    FromAddress = "dexample@gmail.com"
    Subject     = "Sending Email with Attachement Using SendGrid and Powershell Script"
    Body        = "Please find the attachement."
    APIKey       = "SG.Jrasfdasfdafdlernsadf.sdnmflasfdansfdlafnasfdasf50H_SpfvM"
    FileName ="SampleAttachment.html"
    FileNameWithFilePath = "d://1.html"
    AttachementType ="text/html"
}
SendGridMailWithAttachment @Parameters
