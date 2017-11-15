REST API
========

Global Requirements
-------------------

There are a few HTTP headers that are required. Firstly, authentication needs to be in place (see previous article). Secondly, `Content-Type` and `Content-Length` needs to be set to `application/json; utf8` and the size of the body respectively. This makes sure the API understands that you send json formatted and utf8 encoded data in your request body.

Authentication
--------------
All API calls require authentication with your account credentials. The REST API uses [`HTTP Basic Authentication`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication#Basic_authentication_scheme) using your account ID and API key. For how to get API credentials checkout [this page](Authentication).

To get your Basic Authentication string, take your account ID (i. e. `ip1-12345`) and your API-key (i. e. `cCnbXFfyTM5BTKh7uNV`) and run them through a Base64 encoder in the following way:

```
ip1-12345:CnbXFfyTM5BTKh7uNV
```
You should get at string that looks something like this:

```
aXAxLTEyMzQ1OkNuYlhGZnlUTTVCVEtoN3VOVg==
```
This is your HTTP Basic Authentication string and should be given as an HTTP header in the following way:

```
Authorization: Basic aXAxLTEyMzQ1OkNuYlhGZnlUTTVCVEtoN3VOVg==`
```


Sending SMS
-----------
 
 * Base URL: `https://api.ip1sms.com/`
 * Endpoint: `api/sms/send`

In order to send SMS you need to have credits. When you create an account and verify your phone number you'll be given €1 in credits, the amount of SMS you'll be able to send with those credits will heavily depend on what country you'll be sending the SMSes to.

When creating a request to send SMS we use the `/api/sms/send` endpoint with the HTTP method `POST`. 
Since our REST API is JSON based that's the data structure we'll use and when sending SMS we have seven properties to play with:
* `From`: Who or what the SMS should be sent from can be either a up to 15 digit (telephone) number or an up to 11 character ASCII string.
* `Numbers`: A `collection` of phone numbers with country code, eg. 46 for Sweden.
* `Contacts`: A `collection` of contact IDs. See the Contact section for how to use these.
* `Groups`: A `collection` of group IDs. See the Group section for how to use these.
* `Message`: A `string` containing the message you want to send to the recipients above.
* `Prio`: DEPRECATED! This value was previously used for sending SMS with higher priority. Now all SMS are sent with with the same prio independently of what Prio value is used.
* `Email`: A `boolean` value of whether email copies should be sent or not to the recipients that has an email provided previously. Note that in order for email copies to be sent you need to purchase the service `Email Copy`.

All recipients (Numbers, Contacts, Groups) will be converted into a single collection of Contacts (So that emails addresses and templating fields are conserved where applicable) and then remove all duplicated contacts distincted by their phone number.

``` json
{
  "Numbers": [
    "+14155552671",
    "+14155552672"
  ],
  "Contacts": [
    2314,
    2412
  ],
  "Groups": [
    989,
    898
  ],
  "Email": false,
  "Prio": 1,
  "From": "iP.1",
  "Message": "This is the day you will always remember as the day you almost caught Captain Jack Sparrow!"
}
```

We have an automatically generated [site for documentation](https://api.ip1sms.com/Help/Api/POST-api-sms-send) which you can look at for futher details while sending SMS

### Templating

Templating can be done by wrapping an argument with curly brackets (`{lastName}`) and then adding an object to the `Parameters` array with the phone number as the property name. For phone numbers not provided you may also specify templating values in the `Parameters` property `default`. The arguments specified in the message text will be replaced with their respective values prioritized in the following order:

1. Number specific value eg. `"4610606060": {"title": "Sir"}`
2. Default value
3. Empty string

Note that if a you specify an argument in the message but don't provide an example for it in the parameter data it will not be replaced. As you can see below the whole message is wrapped in curly brackets. However if a parameter is given to only one recipient and not the others (including default) the parameter will default to empty string. This enables us to still use curly brackets outside of templating.


``` json
{
  "From": "TestNOS",
  "Numbers": [
    "4610606060",
    "46769447696"
  ],
  "Message": "{: Hello {title} {lastName}, how are you doing this fine day? :}",
  "Parameters": {
    "4610606060": {
      "title": "Sir",
      "lastName": "Newton"
    },
    "default": {
      "lastName": "Friend"
    }
  }
}

```
### Code Examples
Below you'll see example code for sending an SMS in a few languages.
#### .NET / C#
``` csharp
string account = "ip1-xxxxx";
string password = "cCnbXFfyTM5BTKh7uNV"

using (var client = new HttpClient())
{
    client.BaseAddress = new Uri("https://api.ip1sms.com/api/");
    client.DefaultRequestHeaders.Accept.Clear();
    client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    string authString = Convert.ToBase64String(Encoding.ASCII.GetBytes($"{account}:{password}");
    client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", authString);

    var sms = new object()
    {
        From = "iP1",
        Numbers = new List<string>() { "46123456789" },
        Contacts = new List<int>(),
        Groups = new List<int>(),
        Message = "This is the day you will always remember as the day you almost caught Captain Jack Sparrow!",
        Email = false
    };

    HttpResponseMessage response = await client.PostAsJsonAsync("sms/send", sms);
}
```

#### PHP

``` php
// Account information and where we will send the request to
$conf = array(
  'account' => 'ip1-xxxxx',
  'password' => 'cCnbXFfyTM5BTKh7uNV',
  'apiUrl' => 'api.ip1sms.com',
  'method' => 'POST',
  'endpoint' => '/api/sms/send',
);
// The message it self
$message = array(
  'Numbers' => ['4673055555'],
  'From' => 'iP.1',
  'Message' => "This is the day you will always remember as the day you almost caught Captain Jack Sparrow!",
);
// Encode to JSON
$jsonEncodedMessage = json_encode($message);
// Set up request options
$options = array(
    'http' => array(
        'header'  => array(
            'Content-Type: application/json',
            'Authorization: Basic '. base64_encode($conf['account'].":".$conf['password']),
            'Content-Length: ' . strlen($jsonEncodedMessage)
        ),
        'method'  => $conf['method'],
        'content' => $jsonEncodedMessage,
    )
);
// Construct the URL
$url = "https://" . $conf['apiUrl'] . $conf['endpoint'];
$context  = stream_context_create($options);
// Send the request
$response = file_get_contents($url, false, $context);
```