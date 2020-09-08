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