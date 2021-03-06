CollectionFS
============

미티어에 패키지중 하나인 CollectionFS는 파일 업로드, 다운로드, 저장, 동기화,
관리, 그리고 복사등의 기능을 제공한다. 또한, 저장은 로컬 파일, 몽고디비의 GridFS, 아마존 S3 아답터로 지원 한다.추가 저장 아답터들을 개발해서 추가할 수도 있다.

[![Build Status](https://travis-ci.org/CollectionFS/Meteor-CollectionFS.png?branch=master)](https://travis-ci.org/CollectionFS/Meteor-CollectionFS)
[![Join the chat at https://gitter.im/CollectionFS/Meteor-CollectionFS](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/CollectionFS/Meteor-CollectionFS?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Victor Leung wrote a great [quick start guide](https://medium.com/@victorleungtw/how-to-upload-files-with-meteor-js-7b8e811510fa) for the most basic image uploading and displaying task.

[Check out the Wiki](https://github.com/CollectionFS/Meteor-CollectionFS/wiki) for more information, code examples and how-tos.

## 내용 

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [중요 사항](#중요-사항)
- [설치](#설치)
- [소개](#소개)
- [시작하기](#시작하기)
  - [업로드 해보기](#업로드-해보기)
  - [`insert` 제대로 사용하기](#insert-제대로-사용하기)
  - [업로드이후 작업](#업로드이후-작업)
- [스토리지 아답터](#스토리지-아답터)
- [파일 조작하기](#파일-조작하기)
  - [transformWrite / transformRead](#transformwrite--transformread)
  - [beforeWrite](#beforewrite)
- [이미지 조작하기](#이미지-조작하기)
  - [기본 예제](#기본-예제)
  - [최적화](#최적화)
- [필터링](#필터링)
- [FS.File 인스턴스](#fsfile-인스턴스)
  - [Storing FS.File references in your objects](#storing-fsfile-references-in-your-objects)
- [보안](#보안)
  - [Securing Based on User Information](#securing-based-on-user-information)
- [업로드 이미지 표시하기](#업로드-이미지-표시하기)
- [UI Helpers](#ui-helpers)
  - [FS.File Instance Helper Methods](#fsfile-instance-helper-methods)
  - [url](#url)
  - [isImage](#isimage)
  - [isAudio](#isaudio)
  - [isVideo](#isvideo)
  - [isUploaded](#isuploaded)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## 중요 사항

### Cordova Android Bug with Meteor 1.2+

Due to a [bug in the Cordova Android version that is used with Meteor 1.2](https://issues.apache.org/jira/browse/CB-8608?jql=project%20%3D%20CB%20AND%20text%20~%20%22FileReader%22), you will need to add the following to your mobile-config.js or you will have problems with this package on Android devices:

```js
App.accessRule("blob:*");
```

### 문서 피드백

If you have Documentation feedback/requests please post on [issue 206](https://github.com/CollectionFS/Meteor-CollectionFS/issues/206)

## 설치

**Meteor 0.9.0 이상 버전만을 지원**

```bash
$ cd <app dir>
``` 

반드시 메인 패키지인 `cfs:standard-packages`를 설치하여야한다:

```bash
$ meteor add cfs:standard-packages
``` 

반드시 스토리지 아답터중 하나를 설치 하여야한다. 스토리지 아답터 패키지는 아래 리스트와 같다. 비록 사용 하지 않더라도, 적어도 `cfs:gridfs` 나 `cfs:filesystem` 중 하나를 반드시 설치하여야한다. 임시저장소가 그중 하나를 필요로 한다. 

```bash
$ meteor add cfs:gridfs

# OR

$ meteor add cfs:filesystem

# OR

$ meteor add cfs:s3

# OR

$ meteor add cfs:dropbox

# OR

$ meteor add iyyang:cfs-aliyun
``` 

필요에 따라서 추가 애드온 패키지를 설치하여 사용 할수 있다. 이것들은 문서 섹션에 설명되 있다.

```bash
$ meteor add <CFS add-on package name>
``` 

## 소개

CollectionFS 패키지는 2개의 글로벌 변수를 사용 할수 있게한다:
`FS.File` 과 `FS.Collection` 이다.

* `FS.File` 인스턴스는 클라이언트 또는 서버의 파일과 그것의 데이터를 감싼다. 브라우저의 `File` object 와 비슷하다 (그리고 `File` object로 부터 생성할수있다), 하지만 추가 properties 와 methods를 가지고 있다. `find` 와 `findOne`의 호출에 의해 반환된 instance 의 methods 는 reactive 이다.
* `FS.Collection` 컬렉션을 제공한다. 그리고 그 컬렉션은 저장 파일 정보를 가지고 있다. `FS.Collection` 은  `Mongo.Collection` instance에 기반을 두고 있다. 대부분의 컬렉션 메소드들 즉, `find` 나 `insert` 등의 메소드를 `FS.Collection`은 제공한다. 만약  `_ensureIndex` 같은 다른 컬렉션 메소드를 사용하고 싶다면,`Mongo.Collection` instance 에 기반을 둔 `myFSCollection.files` 를  통해서도 가능 하다..

`FS.Collection` 문서는  `FS.File` 로 표현 될수 있다.

CollectionFS 는 HTTP 업로드 패키지를 제공한다. 그것은 파일업로드의 기본 매카니즘을 제공한다. 리엑티브하게 업로드상태를 보는 기능과 일시정지와 업로드 재개 기능을 제공한다. 

## 시작하기

패키지를 사용할때 첫번째 작업은 `FS.Collection` 를 정의 하는것이다.

### FS Collection 과 Filestore 생성하기
*common.js:*

```js
Images = new FS.Collection("images", {
  stores: [new FS.Store.FileSystem("images", {path: "~/uploads"})]
});
```

이 예제에서 "images" 라는 이름의 FS.Collection 을 생성 했다. 컬렉션을 생성하면 서버의 몽고디비 데이터베이스에 "cfs.images.filerecord" 라는 이름으로 컬렉션을 생성된다. 이전에 파일 시스텝 어뎁터를 사용해야 한다고 말했었다. 그리고 아답터는 로커 파일 시스템의 `~/uploads` 경로를 사용한다. 특정 `path`를 지정하지 않으면 앱의 컨테이너 안에 `cfs/files` 라는 이름으로 폴더를 생성하여 사용한다.

FS.Collection 과 FS.Store 변수는 클라이언트와 서버코드 모두 선언할 필요는 없다. 
하지만 서버와 클라이언트 모두 같은 이름을 사용하여 인스턴스를 생성하여야한다(각 생성자의 첫번째 파라메터). 

### 업로드 권한주기 (insert)
사용자가 FS Collection 에 submit 하기위하여, 반드시 서버코드에 `allow` 룰을 생성하자 :
**server.js** or within **Meteor.isServer** block:

```javascript
Images.allow({
  'insert': function () {
    // add custom authentication code here
    return true;
  }
});
```

### 업로드 해보기
이제 클라이언트에서 다음 코드와같이 업로드 할수 있다. 다음 예제는 HTML file input change 이벤트에서 업로드하는 코드이다 :

```js
Template.myForm.events({
  'change .myFileInput': function(event, template) {
    var files = event.target.files;
    for (var i = 0, ln = files.length; i < ln; i++) {
      Images.insert(files[i], function (err, fileObj) {
        // Inserted new doc with ID fileObj._id, and kicked off the data upload using HTTP
      });
    }
  }
});
```
옵션으로, 코드를 좀더 깔끔하게 작성하기 위하여, 제공된 `FS.Utility.eachFile` 이라는 메소드를 사용할수 있다 :

```js
Template.myForm.events({
  'change .myFileInput': function(event, template) {
    FS.Utility.eachFile(event, function(file) {
      Images.insert(file, function (err, fileObj) {
        // Inserted new doc with ID fileObj._id, and kicked off the data upload using HTTP
      });
    });
  }
});
```

여기서 우리가 하는 일은 브라우저가 제공하는 `File` 오브젝트를 `Images.insert()` 에 넣는것 뿐이다. 이것은 `File` 로부터 `FS.File` 을 생성한뒤 `Images` FS.Collection 에  연결한다. 그리고 리엑티브하게 상태를 업데이트 하면서 데이터를 서버로 업로드 한다. 

`insert` 메소드는 첫번째 파라메터로 다양한 형태의 포멧을 가진 파일을 받는다 : 

* `File` object (client only)
* `Blob` object (client only)
* `Uint8Array`
* `ArrayBuffer`
* `Buffer` (server only)
* A full URL that begins with "http:" or "https:"
* A local filepath (server only)
* A data URI string

Where possible, streams are used, so in general you should avoid using any
of the buffer/binary options unless you have no choice, perhaps because you
are generating small files in memory.

The most common usage is to pass a `File` object on the client or a URL on
either the client or the server. Note that when you pass a URL on the client,
the actual data download from that URL happens on the server, so you don't
need to worry about CORS. In fact, we recommend doing all inserts on the
client (managing security through allow/deny), unless you are generating
the data on the server.


### `insert` 제대로 사용하기

When you need to insert a file that's located on a client, always call 
`myFSCollection.insert` on the client. While you could define your own method,
pass it the `fsFile`, and call `myFSCollection.insert` on the server, the
difficulty is with getting the data from the client to the server. When you
pass the fsFile to your method, only the file *info* is sent and not the *data*.

By contrast, when you do the insert directly on the client, it automatically
chunks the file's data after insert, and then queues it to be sent chunk by
chunk to the server. And then there is the matter of recombining all those
chunks on the server and stuffing the data back into the fsFile. So doing
client-side inserts actually saves you all of this complex work, and that's
why we recommend it.

Calling insert on the server should be done only when you have the file
somewhere on the server filesystem already or you're generating the data
on the server.

### 업로드 이후 작업

After the server receives the `FS.File` and all the corresponding binary file
data, it saves copies of the file in the stores that you specified.

If any storage adapters fail to save any of the copies in the
designated store, the server will periodically retry saving them. After a
configurable number of failed attempts at saving, the server will give up.

To configure the maximum number of save attempts, use the `maxTries` option
when creating your store. The default is 5.

## 스토리지 아답터

Storage adapters handle retrieving the file data and removing the file data
when you delete the file. There are currently four available storage adapters, which are in separate
packages. Refer to the package documentation for usage instructions.

* [cfs:gridfs](https://github.com/CollectionFS/Meteor-CollectionFS/tree/devel/packages/gridfs): Allows you to save data to mongodb GridFS.
* [cfs:filesystem](https://github.com/CollectionFS/Meteor-CollectionFS/tree/devel/packages/filesystem): Allows you to save to the server filesystem.
* [cfs:s3](https://github.com/CollectionFS/Meteor-CollectionFS/tree/devel/packages/s3): Allows you to save to an Amazon S3 bucket.
* [cfs:dropbox](https://github.com/CollectionFS/Meteor-CollectionFS/tree/devel/packages/dropbox): Allows you to save to a Dropbox account.
* [iyyang:cfs-aliyun](https://github.com/yyang/cfs-aliyun): Allows you to save to Aliyun OSS Storage.
 
### Securing sensetive information
_If you're using a storage adapter that requires sensitive information such as
access keys, we recommend supplying that information using environment variables.
If you instead decide to pass options to the storage adapter constructor,
then be sure that you do that only in the server code (and not simply within a
`Meteor.isServer` block)._

## File Manipulation
 
You may want to manipulate files before saving them. For example, if a user
uploads a large image, you may want to reduce its resolution, crop it,
compress it, etc. before allowing the storage adapter to save it. You may also
want to convert to another content type or change the filename or encrypt
the file. You can do all of this by defining stream transformations on a
store.

> Note: Transforms only work on the server-side code

### transformWrite / transformRead

The most common type of transformation is a "write" transformation, that is,
a function that changes the data as it is initially stored. You can define
this function using the `transformWrite` option on any store constructor. If the
transformation requires a companion transformation when the data is later read
out of the store (such as encrypt/decrypt), you can define a `transformRead`
function as well.

For illustration purposes, here is an example of a `transformWrite` function that doesn't do anything:

```js
transformWrite: function(fileObj, readStream, writeStream) {
  readStream.pipe(writeStream);
}
```

The important thing is that you must pipe the `readStream` to the `writeStream` before returning from the function. Generally you will manipulate the stream in some way before piping it.

### beforeWrite

Sometimes you also need to change a file's metadata before it is saved to a particular store. For example, you might have a `transformWrite` function that changes the file type, so you need a `beforeWrite` function that changes the extension and content type to match.

The simplest type of `beforeWrite` function will return an object with `extension`, `name`, or `type` properties. For example:

```js
beforeWrite: function (fileObj) {
  return {
    extension: 'jpg',
    type: 'image/jpg'
  };
}
```

This would change the extension and type for that particular store.

Since `beforeWrite` is passed the `fileObj`, you can optionally alter that directly. For example, the following would be the same as the previous example assuming the store name is "jpegs":

```js
beforeWrite: function (fileObj) {
  fileObj.extension('jpg', {store: "jpegs", save: false});
  fileObj.type('image/jpg', {store: "jpegs", save: false});
}
```

(It's best to provide the `save: false` option to any of the setters you call in `beforeWrite`.) 

## 이미지 조작하기

A common use for `transformWrite` is to manipulate images before saving them.
To get this set up:

1. Install [GraphicsMagick](http://www.graphicsmagick.org/) or [ImageMagick](http://www.imagemagick.org/script/index.php) on your development machine and on any server that will host your app. (The free Meteor deployment servers do not have either of these, so you can't deploy to there.) These are normal operating system applications, so you have to install them using the correct method for your OS. For example, on Mac OSX you can use `brew install graphicsmagick` assuming you have Homebrew installed.
2. Add the `cfs:graphicsmagick` Meteor package to your app: `meteor add cfs:graphicsmagick`

### 기본 예제

```js
var createThumb = function(fileObj, readStream, writeStream) {
  // Transform the image into a 10x10px thumbnail
  gm(readStream, fileObj.name()).resize('10', '10').stream().pipe(writeStream);
};

Images = new FS.Collection("images", {
  stores: [
    new FS.Store.FileSystem("thumbs", { transformWrite: createThumb }),
    new FS.Store.FileSystem("images"),
  ],
  filter: {
    allow: {
      contentTypes: ['image/*'] //allow only images in this FS.Collection
    }
  }
});
```

Check out the [Wiki](https://github.com/CollectionFS/Meteor-CollectionFS/wiki) for more examples and How-tos.

### 최적화

* When you insert a file, a worker begins saving copies of it to all of the
stores you define for the collection. The copies are saved to stores in the
order you list them in the `stores` option array. Thus, you may want to prioritize
certain stores by listing them first. For example, if you have an images collection
with a thumbnail store and a large-size store, you may want to list the thumbnail
store first to ensure that thumbnails appear on screen as soon as possible after
inserting a new file. Or if you are storing audio files, you may want to prioritize
a "sample" store over a "full-length" store.


## 필터링

You may specify filters to allow (or deny) only certain content types,
file extensions or file sizes in a FS.Collection with the `filter` option:

```js
Images = new FS.Collection("images", {
  filter: {
    maxSize: 1048576, // in bytes
    allow: {
      contentTypes: ['image/*'],
      extensions: ['png']
    },
    deny: {
      contentTypes: ['image/*'],
      extensions: ['png']
    },
    onInvalid: function (message) {
      if (Meteor.isClient) {
        alert(message);
      } else {
        console.log(message);
      }
    }
  }
});
```

Alternatively, you can pass your filters object to `myFSCollection.filters()`.

To be secure, this must be added on the server. However, you should use the `filter`
option on the client, too, to help catch many of the disallowed uploads there
and allow you to display a helpful message with your `onInvalid` function.

You can mix and match filtering based on extension or content types.
The contentTypes array also supports "image/\*" and "audio/\*" and "video/\*"
like the "accepts" attribute on the HTML5 file input element.

If a file extension or content type matches any of those listed in allow,
it is allowed. If not, it is denied. If it matches both allow and deny,
it is denied. Typically, you would use only allow or only deny,
but not both. If you do not pass the `filter` option, all files are allowed,
as long as they pass the tests in your `FS.Collection.allow()` and
`FS.Collection.deny()` functions.

The extension checks are used only when there is a filename. It's possible to
upload a file with no name. Thus, you should generally use extension checks
only *in addition to* content type checks, and not instead of content type checks.

The file extensions must be specified without a leading period. Extension matching
is case-insensitive.

## FS.File 인스턴스

An `FS.File` instance is an object with properties similar to this:

```js
{
  _id: '',
  collectionName: '', // this property not stored in DB
  collection: collectionInstance, // this property not stored in DB
  createdByTransform: true, // this property not stored in DB
  data: data, // this property not stored in DB
  original: {
    name: '',
    size: 0,
    type: '',
    updatedAt: date 
  },
  copies: {
    storeName: {
      key: '',
      name: '',
      size: 0,
      type: '',
      createdAt: date,
      updatedAt: date 
    }
  },
  uploadedAt: date,
  anyUserDefinedProp: anything
}
```

But `name`, `size`, `type`, and `updatedAt` should be retrieved and set with the methods rather than directly accessing the props:

```js
// get original
fileObj.name();
fileObj.extension();
fileObj.size();
fileObj.formattedSize(); // must add the "numeral" package to your project to use this method
fileObj.type();
fileObj.updatedAt();

// get for the version in a store
fileObj.name({store: 'thumbs'});
fileObj.extension({store: 'thumbs'});
fileObj.size({store: 'thumbs'});
fileObj.formattedSize({store: 'thumbs'}); // must add the "numeral" package to your project to use this method
fileObj.type({store: 'thumbs'});
fileObj.updatedAt({store: 'thumbs'});

// set original
fileObj.name('pic.png');
fileObj.extension('png');
fileObj.size(100);
fileObj.type('image/png');
fileObj.updatedAt(new Date);

// set for the version in a store
fileObj.name('pic.png', {store: 'thumbs'});
fileObj.extension('png', {store: 'thumbs'});
fileObj.size(100, {store: 'thumbs'});
fileObj.type('image/png', {store: 'thumbs'});
fileObj.updatedAt(new Date, {store: 'thumbs'});
```

These methods can all be used as UI helpers, too:

```html
{{#each myFiles}}
  <p>Original name: {{this.name}}</p>
  <p>Original extension: {{this.extension}}</p>
  <p>Original type: {{this.type}}</p>
  <p>Original size: {{this.size}}</p>
  <p>Thumbnail name: {{this.name store="thumbs"}}</p>
  <p>Thumbnail extension: {{this.extension store="thumbs"}}</p>
  <p>Thumbnail type: {{this.type store="thumbs"}}</p>
  <p>Thumbnail size: {{this.size store="thumbs"}}</p>
{{/each}}
```

Also, rather than setting the `data` property directly, you should use the `attachData` method.

[Check out the full public API for `FS.File`](https://github.com/CollectionFS/Meteor-CollectionFS/blob/devel/packages/file/api.md).

### Storing FS.File references in your objects

**_NOTE:_**
_At the moment storing FS.File - References in MongoDB on the server side doesn't work. See eg. (https://github.com/CollectionFS/Meteor-cfs-ejson-file/issues/1) (https://github.com/CollectionFS/Meteor-CollectionFS/issues/356)
(https://github.com/meteor/meteor/issues/1890)._

_Instead store the _id's of your file objects and then fetch the FS.File-Objects from your CollectionFS - Collection._

Often your files are part of another entity. You can store a reference to the file directly in the entity.
You need to add `cfs:ejson-file` to your packages with `meteor add cfs:ejson-file`.
Then you can do for example:

```js
// Add file reference of the event photo to the event
var file = $('#file').get(0).files[0];
var fileObj = eventPhotos.insert(file);
events.insert({
  name: 'My Event',
  photo: fileObj
});

// Later: Retrieve the event with the photo
var event = events.findOne({name: 'My Event'});
// This loads the data of the photo into event.photo
// You can include it in your collection transform function.
event.photo.getFileRecord();
```

[Demo app](https://github.com/Sanjo/collectionFS_test/tree/ejson-file-reference)

You need to ensure that the client is subscribed to the related photo document, too.
There are packages on atmosphere, such as
[publish-with-relations](https://atmospherejs.com/package/publish-with-relations) and
[smart-publish](https://atmospherejs.com/package/smart-publish), that attempt to make this easy.

In simple cases, you may also be able to return an array from [Meteor.publish](http://docs.meteor.com/#/full/meteor_publish):

```javascript
Meteor.publish("memberAndPhotos", function (userId) {
  check(userId, String);
  return [
    Collections.Members.find({userId: userId}, {fields: {secretInfo: 0}}),
    Collections.Photos.find({
      $query: {'metadata.owner': userId},
      $orderby: {uploadedAt: -1}
    });
  ];
});
```

## 보안

File uploads and downloads can be secured using standard Meteor `allow`
and `deny` methods. To best understand how CollectionFS security works, you
must first understand that there are two ways in which a user could interact
with a file:

* She could view or edit information *about* the file or any
custom metadata you've attached to the file record.
* She could view or edit the *actual file data*.

You may find it necessary to secure file records with different criteria
from that of file data. This is easy to do.

Here's an overview of the various ways of securing various aspects of files:

* To determine who can *see* file metadata, such as filename, size, content type,
and any custom metadata that you set, use normal Meteor publish/subscribe
to publish and subscribe to an `FS.Collection` cursor. This does not allow the
user to *download* the file data.
* To determine who can *download* the actual file, use "download" allow/deny
functions. This is a custom type of allow/deny function provided by CollectionFS.
The first argument is the userId and the second argument is the FS.File being
requested for download. An example:
```
Images.allow({
	download: function(userId, fileObj) {
		return true
	}
})
```
* To determine who can *set* file metadata, insert files, and upload file data,
use "insert" allow/deny functions.
* To determine who can *update* file metadata, use "update" allow/deny functions.
* To determine who can *remove* files, which removes all file data and file
metadata, use "remove" allow/deny functions.

The `download` allow/deny functions can be thought of essentially as allowing or
denying "read" access to the file. For a normal Meteor collection, "read" access
is defined through pub/sub, but we don't want to send large amounts of binary file
data to each client just because they subscribe to the file record. Thus with CFS,
pub/sub will get you the file's metadata on the client whereas an HTTP request to the
GET URL is required to view or download the file itself. The `download` allow/deny
determines whether this HTTP request will respond with "Access Denied" or not.

### Securing Based on User Information

To secure a file based on a user "owner" or "role" or some other piece of custom
metadata, you must set this information on the file when originally inserting it.
You can then check it in your allow/deny functions.

```js
var fsFile = new FS.File(event.target.files[0]);
fsFile.owner = Meteor.userId();
fsCollection.insert(fsFile, function (err) {
  if (err) throw err;
});
```

Note that you will want to verify this `owner` metadata in a `deny` function
since the client could put any user ID there.

## 업로드 이미지 표시하기

Create a helper that returns your image files:

```js
Template.imageView.helpers({
  images: function () {
    return Images.find(); // Where Images is an FS.Collection instance
  }
});
```

Use the `url` method with an `img` element in your markup:

```html
<template name="imageView">
  <div class="imageView">
    {{#each images}}
      <div>
        <a href="{{this.url}}" target="_blank"><img src="{{this.url store='thumbs' uploading='/images/uploading.gif' storing='/images/storing.gif'}}" alt="" class="thumbnail" /></a>
      </div>
    {{/each}}
  </div>
</template>
```

Notes:
* `{{this.url}}` will assume the first store in your `stores` array. In this example, we're displaying the image from the "thumbs" store but wrapping it in a link that will load the image from the primary store (for example, the original image or a large image).
* The `uploading` and `storing` options allow you to specify a static image that will be shown in place of the real image while it is being uploaded and stored. You can alternatively use `if` blocks like `{{#if this.isUploaded}}` and `{{#if this.hasStored 'thumbs'}}` to display something different until upload and storage is complete.
* These helpers are actually just instance methods on the `FS.File` instances, so there are others you can use, such as `this.isImage`. See [the API documentation](https://github.com/CollectionFS/Meteor-cfs-file/blob/master/api.md). The `url` method is documented separately [here](https://github.com/CollectionFS/Meteor-cfs-access-point/blob/master/api.md#fsfileurloptionsanywhere).



## UI Helpers

There are two types of UI helpers available. First, some of the `FS.File`
instance methods will work when called in templates, too. These are available
to you automatically and are documented here. Second, some additional useful helpers are provided
in the optional [cfs-ui](https://github.com/CollectionFS/Meteor-cfs-ui) package.
These make it easy to render a delete button or an upload progress bar
and more. Refer to the `cfs-ui` readme.

### FS.File Instance Helper Methods

Some of the FS.File API methods are designed to be usable as UI helpers.

### url

Returns the HTTP file URL for the current FS.File.

Use with an `FS.File` instance as the current context.

Specify a `store` attribute to get the URL for a specific store. If you don't
specify the store name, the URL will be for the copy in the first defined store.

```html
{{#each images}}
  URL: {{this.url}}
  <img src="{{this.url store='thumbnail'}}" alt="thumbnail">
{{/each}}
```

This is actually using the [url method](https://github.com/CollectionFS/Meteor-cfs-access-point/blob/master/api.md#fsfileurloptionsanywhere), which is added to the `FS.File` prototype by the `cfs:access-point` package. You can use any of the options mentioned in the API documentation, and you can call it from client and server code.

### isImage

Returns true if the copy of this file in the specified store has an image
content type. If the file object is unmounted or was not saved in the specified
store, the content type of the original file is checked instead.

Use with an `FS.File` instance as the current context.

```html
{{#if isImage}}
{{/if}}
{{#if isImage store='thumbnail'}}
{{/if}}
```

### isAudio

Returns true if the copy of this file in the specified store has an audio
content type. If the file object is unmounted or was not saved in the specified
store, the content type of the original file is checked instead.

Use with an `FS.File` instance as the current context.

```html
{{#if isAudio}}
{{/if}}
{{#if isAudio store='thumbnail'}}
{{/if}}
```

### isVideo

Returns true if the copy of this file in the specified store has a video
content type. If the file object is unmounted or was not saved in the specified
store, the content type of the original file is checked instead.

Use with an `FS.File` instance as the current context.

```html
{{#if isVideo}}
{{/if}}
{{#if isVideo store='thumbnail'}}
{{/if}}
```

### isUploaded

Returns true if all the data for the file has been successfully received on the
server. It may not have been stored yet.

Use with an `FS.File` instance as the current context.

```html
{{#with fileObj}}
{{#if isUploaded}}
{{/if}}
{{/with}}
```
