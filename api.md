
#### <a name="FS.File.prototype.reload"></a>FS.File.prototype.reload {any}&nbsp;&nbsp;<sub><i>Anywhere</i></sub> ####
@FS.File.prototype.reload Updates the "in FS.File instance object"
-
> __Warning!__
> This property "FS.File.prototype.reload" has deprecated from the api
> We should not maintain duplicate data
> This function is deprecating - but we cannot remove it before all
> references are updated to use `FS.File.fetch()`

> ```FS.File.prototype.reload = function() { ...``` [fsFile/fsFile-common.js:68](fsFile/fsFile-common.js#L68)

-

-
Since the client can't block and we need to update self after being
sure the update went through, we need a callback

-
Client: Instructs the DownloadTransferQueue to begin downloading the file copy
Server: Returns the Buffer data for the copy

#### <a name="FS.File.prototype.url"></a>FS.File.prototype.url([options], [auth], [download])&nbsp;&nbsp;<sub><i>Anywhere</i></sub> ####
@FS.File.prototype.url Construct the file url
-

__Arguments__

* __options__ *{object}*    (Optional)    - __copy__ *{string}*    (Default = "_master")
The copy of the file to get
* __auth__ *{boolean}*    (Optional = null)
Wether or not the authenticate
* __download__ *{boolean}*    (Optional = true)
Should headers be set to force a download

-

> ```FS.File.prototype.url = function(options) { ...``` [fsFile/fsFile-common.js:193](fsFile/fsFile-common.js#L193)

-

-
Return the http url for getting the file - on server set auth if wanting to
use authentication on client set auth to true or token

-
check for "hash" prop if called as helper
We check if the copy is found

#### <a name="FS.File.prototype.downloadUrl"></a>FS.File.prototype.downloadUrl([options], [auth])&nbsp;&nbsp;<sub><i>Anywhere</i></sub> ####
@FS.File.prototype.downloadUrl Get the download url
-
> __Warning!__
> This method "FS.File.prototype.downloadUrl" has deprecated from the api
> Use The hybrid helper `FS.File.url`

__Arguments__

* __options__ *{object}*    (Optional)    - __copy__ *{string}*    (Default = "_master")
The copy of the file to get
* __auth__ *{boolean}*    (Optional = null)
Wether or not the authenticate

-

> ```FS.File.prototype.downloadUrl = function(options) { ...``` [fsFile/fsFile-common.js:244](fsFile/fsFile-common.js#L244)

-

#### <a name="FS.File.prototype.put"></a>FS.File.prototype.put([callback])&nbsp;&nbsp;<sub><i>Anywhere</i></sub> ####
-

__Arguments__

* __callback__ *{function}*    (Optional)
Callback for returning errors and id

-
```
fo.put(function(err, id) {
   if (err) {
     console.log('Got an error');
   } else {
     console.log('Passed on the file id: ' + id);
   }
 });
```

> ```FS.File.prototype.put = function(callback) { ...``` [fsFile/fsFile-common.js:264](fsFile/fsFile-common.js#L264)

-

-
Force bytesUploaded to be equal to the file size in case
this was a server insert or a non-chunked client upload.

#### <a name="FS.File.prototype.getExtension"></a>FS.File.prototype.getExtension()&nbsp;&nbsp;<sub><i>Anywhere</i></sub> ####
@FS.File.prototype.getExtension Returns the file extension
-

__Returns__  *{string | null}*
The extension eg.: `jpg`

> ```FS.File.prototype.getExtension = function() { ...``` [fsFile/fsFile-common.js:299](fsFile/fsFile-common.js#L299)

-

#### <a name="FS.File.prototype.fetch"></a>FS.File.prototype.fetch()&nbsp;&nbsp;<sub><i>Anywhere</i></sub> ####
-

__Returns__  *{object}*
The filerecord

> ```FS.File.prototype.fetch = function() { ...``` [fsFile/fsFile-common.js:373](fsFile/fsFile-common.js#L373)

-

#### <a name="FS.File.prototype.hasCopy"></a>FS.File.prototype.hasCopy(copyName, optimistic)&nbsp;&nbsp;<sub><i>Anywhere</i></sub> ####
-

__Arguments__

* __copyName__ *{string}*  
Name of the copy to check for
* __optimistic__ *{boolean}*  
In case that the file record is not found, read below

-

__Returns__  *{boolean}*
If the copy exists or not
> Note: If the file is not published to the client or simply not found:
> this method cannot know for sure if it exists or not. The `optimistic`
> param is the boolean value to return. Are we `optimistic` that the copy
> could exist. This is the case in `FS.File.url` we are optimistic that the
> copy supplied by the user exists.

> ```FS.File.prototype.hasCopy = function(copyName, optimistic) { ...``` [fsFile/fsFile-common.js:391](fsFile/fsFile-common.js#L391)

-


---

#### <a name="FS.Collection.acceptDropsOn"></a>FS.Collection.acceptDropsOn(templateName, selector, [metadata])&nbsp;&nbsp;<sub><i>Client</i></sub> ####
-

__Arguments__

* __templateName__ *{string}*  
Name of template to apply events on
* __selector__ *{string}*  
The element selector eg. "#uploadField"
* __metadata__ *{object|function}*    (Optional)
Data/getter to attach to the file objects

-
Using this method adds an `uploaded` and `uploadFailed` event to the
template events. The event object contains `{ error, file }`
Example:
```css
.dropzone {
 border: 2px dashed silver; 
 height: 5em;
 padding-top: 3em;
 -webkit-border-radius: 8px;
 -moz-border-radius: 8px;
 -ms-border-radius: 8px;
 -o-border-radius: 8px;
 border-radius: 8px;
}
```
```html
<template name="hello">
Choose file to upload:<br/>
<div id="dropzone" class="dropzone">
<div style="text-align: center; color: gray;">Drop file to upload</div>
</div>
</template>
```
```js
Template.hello.events({
   'uploaded #dropzone': function(event, temp) {
     console.log('Event Uploaded: ' + event.file._id);
   }
 });
images.acceptDropsOn('hello', '#dropzone');
```

> ```FS.Collection.prototype.acceptDropsOn = function(templateName, selector, metadata) { ...``` [fsCollection/api.client.js:99](fsCollection/api.client.js#L99)

-

#### <a name="FS.Collection.acceptUploadFrom"></a>FS.Collection.acceptUploadFrom(templateName, selector, [metadata])&nbsp;&nbsp;<sub><i>Client</i></sub> ####
-

__Arguments__

* __templateName__ *{string}*  
Name of template to apply events on
* __selector__ *{string}*  
The element selector eg. "#uploadField"
* __metadata__ *{object|function}*    (Optional)
Data/getter to attach to the file objects

-
Using this method adds an `uploaded` and `uploadFailed` event to the
template events. The event object contains `{ error, file }`
Example:
```html
<template name="hello">
Choose file to upload:<br/>
<input type="file" id="files" multiple/>
</template>
```
```js
Template.hello.events({
   'uploaded #files': function(event, temp) {
     console.log('Event Uploaded: ' + event.file._id);
   }
 });
images.acceptUploadFrom('hello', '#files');
```

> ```FS.Collection.prototype.acceptUploadFrom = function(templateName, selector, metadata) { ...``` [fsCollection/api.client.js:154](fsCollection/api.client.js#L154)

-