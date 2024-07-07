# push-notification 
## Firebase Cloud Messaging API (HTTP v1) and send push notifications from your ASP.NET Core 6 application.

### Install the required NuGet packages:
- dotnet add package Google.Apis.Auth
- dotnet add package Google.Apis.FirebaseCloudMessaging.v1
### Create a service to handle the FCM HTTP v1 requests
- Create a new service class FcmLogic.cs:
```
using Google.Apis.Auth.OAuth2;
using Google.Apis.FirebaseCloudMessaging.v1;
using Google.Apis.FirebaseCloudMessaging.v1.Data;
using Google.Apis.Services;
using System.IO;
using System.Threading.Tasks;

public class FcmLogic
{
    private readonly FirebaseCloudMessagingService _fcmLogic;

    public FcmLogic()
    {
        var credential = GoogleCredential.FromFile("path/to/your/service-account-file.json");
        _fcmLogic = new FirebaseCloudMessagingService(new BaseClientService.Initializer()
        {
            HttpClientInitializer = credential
        });
    }

    public async Task SendNotificationAsync(string deviceToken, string title, string body)
    {
        var message = new Message
        {
            Token = deviceToken,
            Notification = new Notification
            {
                Title = title,
                Body = body
            }
        };

        var request = _fcmLogic.Projects.Messages.Send(new SendMessageRequest { Message = message }, "projects/YOUR_PROJECT_ID");
        var response = await request.ExecuteAsync();

        if (response == null)
        {
            throw new Exception("Failed to send FCM notification.");
        }
    }
}
```
### appsettings.json
```
  {
  "Fcm": {
    "ServiceAccountPath": "path/to/your/service-account-file.json",
    "ProjectId": "YOUR_PROJECT_ID"
  }
}
```
### In Program.cs 
```
builder.Services.AddSingleton<FcmService>();
```
### Create a new model NotificationRequest
```
public class NotificationRequest
{
    public string DeviceToken { get; set; }
    public string Title { get; set; }
    public string Body { get; set; }
}
```
### Create a new controller NotificationController.cs:
```
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;
[ApiController]
[Route("api/[controller]")]
public class NotificationController : ControllerBase
{
    private readonly FcmLogic _fcmLogic;

    public NotificationController(FcmLogic fcmLogic)
    {
        _fcmLogic = fcmLogic;
    }

    [HttpPost]
    public async Task<IActionResult> SendNotification([FromBody] NotificationRequest request)
    {
        await _fcmLogic.SendNotificationAsync(request.DeviceToken, request.Title, request.Body);
        return Ok(new { message = "Notification sent successfully" });
    }
}
```
