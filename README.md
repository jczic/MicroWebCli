## A micro HTTP Web client for MicroPython (used on ESP32 and [Pycom](http://www.pycom.io) modules)

![HC²](hc2.png "HC²")

Very easy to integrate and very light with only one file :
- `"microWebCli.py"`

Simple but effective :
- Use it to requesting fastly HTTP endpoints
- Supports URL parts and modifiable query string parameters
- Supports secured connections *SSL/TLS over HTTP* (https)
- Supports authentications *HTTP Basic* and *Bearer Token*
- Supports large request and response datas
- Supports form-urlencoded datas on request
- Supports json objects on request and response
- Coulds save to a file the response content of any request
- Adds fast methods to directly GET, POST, JSON and downloads
- Automatically bounces (fast methods) on HTTP(S) redirections (ressources moved)

### Using *microWebCli* fast methods (statics, without classes) :

| Name  | Function |
| - | - |
| Makes a GET request | `MicroWebCli.GETRequest(url, queryParams=None, auth=None, connTimeoutSec=None)` |
| Makes a POST request | `MicroWebCli.POSTRequest(url, formData={}, auth=None, connTimeoutSec=None)` |
| Makes a JSON GET/POST request | `MicroWebCli.JSONRequest(url, o=None, auth=None, connTimeoutSec=None)` |
| Save response of a GET request to file | `MicroWebCli.FileRequest(url, filepath, progressCallback=None, auth=None, connTimeoutSec=None)` |

**`Note` : All fast methods automatically bounces on HTTP(S) redirections**

**`Warning` : Entire content of the response is allocated in memory with GETRequest, POSTRequest and JSONRequest**

### Fast methods usage example :
```python
from microWebCli import MicroWebCli

contentBytes = MicroWebCli.GETRequest('http://my-web-site.io/test.html', { 'pet':'cat' })
print(contentBytes)

contentBytes = MicroWebCli.POSTRequest('http://my-web-site.io/post.php', { 'id':12345 })
print(contentBytes)

auth   = MicroWebCli.AuthBasic('toto', 'blah123')
result = MicroWebCli.JSONRequest('http://my-web-site.io/my-api/user/12', auth=auth)
print('User name is %s %s' % (result.firstname, result.lastname))

def progressCallback(microWebCli, progressSize, totalSize) :
  if totalSize :
    print('Progress: %d bytes of %d downloaded...' % (progressSize, totalSize))
  else :
    print('Progress: %d bytes downloaded...' % progressSize)
filename    = '/flash/test.pdf'
contentType = MicroWebCli.FileRequest('http://my-web-site.io/test.pdf', filename, progressCallback)
print('File of content type "%s" was saved to "%s"' % (contentType, filename))
```

### Using *microWebCli* authentication classes :

| Authentication  | Class constructor |
| - | - |
| Basic HTTP | `auth = MicroWebCli.AuthBasic(user, password)` |
| Bearer Token | `auth = MicroWebCli.AuthToken(token)` |


### Using *microWebCli* main class :

| Name  | Function |
| - | - |
| Constructor | `wCli = MicroWebCli(url='', method='GET', auth=None, connTimeoutSec=None)` |
| Opens HTTP request | `wCli.OpenRequest(data=None, contentType=None, contentLength=None)` |
| Opens HTTP form-urlencoded request | `wCli.OpenRequestFormData(formData={})` |
| Opens HTTP json object request | `wCli.OpenRequestJSONData(o=None)` |
| Writes binary data to opened request | `wCli.RequestWriteData(data)` |
| Gets response class for opened request | `wCli.GetResponse()` |
| Checks if no request is opened | `wCli.IsClosed()` |
| Closes an opened request | `wCli.Close()` |

| Name  | Property |
| - | - |
| Gets or sets the connection timeout in secondes | `wCli.ConnTimeoutSec` |
| Gets or sets the Web method verb | `wCli.Method` |
| Gets or sets the complete Web URL | `wCli.URL` |
| Gets or sets the Web proto (http/https) | `wCli.Proto` |
| Gets or sets the Web host | `wCli.Host` |
| Gets or sets the Web port number | `wCli.Port` |
| Gets or sets the Web path part | `wCli.Path` |
| Gets or sets the Web complete query string | `wCli.QueryString` |
| Gets or sets the dict query parameters | `wCli.QueryParams` |
| Gets or sets the dict headers | `wCli.Headers` |
| Gets or sets the authentication class | `wCli.Auth` |

### Using *microWebCli* response class :

| Name  | Function |
| - | - |
| Gets the client microWebCli class | `resp.GetClient()` |
| Gets the HTTP server address  | `resp.GetAddr()` |
| Gets the HTTP server IP address | `resp.GetIPAddr()` |
| Gets the HTTP server port number | `resp.GetPort()` |
| Gets the HTTP server version | `resp.GetHTTPVersion()` |
| Gets the response status code | `resp.GetStatusCode()` |
| Gets the response status message | `resp.GetStatusMessage()` |
| Checks if it is a success response | `resp.IsSuccess()` |
| Checks if it is a location moved response | `resp.IsLocationMoved()` |
| Gets the location moved URL | `resp.LocationMovedURL()` |
| Gets the dict response headers | `resp.GetHeaders()` |
| Gets the reponse content type | `resp.GetContentType()` |
| Gets the response content length | `resp.GetContentLength()` |
| Reads response content (part or total) | `resp.ReadContent(size=None)` |
| Reads response content into buffer | `resp.ReadContentInto(buf, nbytes=None)()` |
| Reads response content as json object | `resp.ReadContentAsJSON()` |
| Writes response content to a file | `resp.WriteContentToFile(filepath, progressCallback=None)` |
| Checks if response is closed | `resp.IsClosed()` |
| Closes response | `resp.Close()` |


### Example of GET (large response) :
```python
from microWebCli import MicroWebCli

wCli = MicroWebCli('http://my-web-site.io/test.html')
wCli.QueryParams['pet'] = 'cat'
print('GET %s' % wCli.URL)
wCli.OpenRequest()
buf  = memoryview(bytearray(1024))
resp = c.GetResponse()
if resp.IsSuccess() :
  while not resp.IsClosed() :
    x = resp.ReadContentInto(buf)
    if x < len(buf) :
      buf = buf[:x]
    print(buf.decode())
  print('GET success with "%s" content type' % resp.GetContentType())
else :
  print('GET return %d code (%s)' % (resp.GetStatusCode(), resp.GetStatusMessage()))
```

### Example of PUT (large request) :
```python
from microWebCli import MicroWebCli

auth = MicroWebCli.AuthToken('Dk23JHsI7NsMcY3C=')
wCli = MicroWebCli('http://my-web-site.io/my-api/user/12', 'PUT', auth)
print('PUT %s' % wCli.URL)
wCli.OpenRequest('application/octet-stream', fileSize)
while True :
  data = fileObject.read(1024)
  if not data :
    break
  wCli.RequestWriteData(data)
resp = c.GetResponse()
if resp.IsSuccess() :
  o = resp.ReadContentAsJSON()
  print('PUT success')
else :
  print('PUT return %d code (%s)' % (resp.GetStatusCode(), resp.GetStatusMessage()))
```

### Example of saving response to a file :
```python
from microWebCli import MicroWebCli

def progressCallback(microWebCli, progressSize, totalSize) :
  if totalSize :
    print('Progress: %d bytes of %d downloaded...' % (progressSize, totalSize))
  else :
    print('Progress: %d bytes downloaded...' % progressSize)

filename = '/flash/test.pdf'
wCli = MicroWebCli('http://my-web-site.io/test.pdf')
print('GET file %s' % wCli.URL)
wCli.OpenRequest()
resp = c.GetResponse()
contentType = resp.WriteContentToFile(filename, progressCallback)
print('File of content type "%s" was saved to "%s"' % (contentType, filename))
```


### By JC`zic for [HC²](https://www.hc2.fr) ;')

*Keep it simple, stupid* :+1:
