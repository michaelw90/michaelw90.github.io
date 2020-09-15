---
layout: post
title:  "JS AJAX File Upload with Progress"
author: michael
categories: [ development ]
thumb: assets/images/posts/codular-posts.jpg
comments: false
excerpt: "Uploading a file using AJAX, and displaying the progress to your users is a great way to enhance your user's experience while on your website."
hidden: true
---

#### Preface

Nowadays, people love doing things without leaving the page they're viewing, this is mostly done using AJAX. Many times people use jQuery to make it easier, but with the advances in browsers, there's no real need for this. Here we'll be looking at how to upload a file to the server without leaving the page, we'll be using the same backend PHP code that was used in one of our [earlier articles](http://codular.com/php-file-uploads). 

The script will upload a file to the server, show a progress bar highlighting how much has been completed, and then finally return a link to the uploaded file. In some cases, you might want to return an id of the file that you uploaded, or something else for your application.

**Note:** This isn't supported by any older IE browsers, and according to [Can I use](http://caniuse.com/xhr2) we'll be looking to support only IE10+.

#### Let's Code

We'll start with the HTML structure, then the JavaScript, then I'll give you the PHP code, which will be adapted from the aforementioned tutorial - there won't be much explanation of the PHP code.

##### HTML

We'll simply be using two input boxes, one which is of type `file`, and the other which is just a `button` so that we can listen for it to be clicked to send the File Upload request. We'll also have a div that we alter the width of to highlight the status of the upload.

It's as simple as this: 

```html
<!doctype html>
<html>
<head>
	<meta charset="utf-8">
	<title>JS File Upload with Progress</title>
	<style>
	.container {
	    width: 500px;
	    margin: 0 auto;
	}
	.progress_outer {
	    border: 1px solid #000;
	}
	.progress {
	    width: 20%;
	    background: #DEDEDE;
	    height: 20px;  
	}
	</style>
</head>
<body>
	<div class='container'>
		<p>
			Select File: <input type='file' id='_file'> <input type='button' id='_submit' value='Upload!'>
		</p>
		<div class='progress_outer'>
			<div id='_progress' class='progress'></div>
		</div>
	</div>
	<script src='upload.js'></script>
</body>
</html>
```

You'll see that we've got a bit of styling on the progress bar, and also added the script tag at the bottom to handle the upload, and progress also.


##### JavaScript

First, we need to make sure that we've got the elements that we're going to use, this is the three elements that have ids. 
	
```javascript
var _submit = document.getElementById('_submit'), 
_file = document.getElementById('_file'), 
_progress = document.getElementById('_progress');
```

Next, add an event listener to the click event of the `_submit` button, which will initiate the upload of the file that's been selected. For this, we'll simply use the `addEventListener` method, and we'll make it call a function called `upload`.

```javascript
_submit.addEventListener('click', upload);
```

Now we can move on to handling the upload, there are a few steps to this: 

1. Check that a file has been selected
2. Dynamically create the form data to send
3. Create an XMLHttpRequest to the upload script with event listeners
4. Send the upload

###### Check that a file has been selected

Our `_file` input has a parameter called `files` which is an array of the files that have been selected - there will be more than one if you set the `multiple` parameter. We can simply check if the `length` of this array is greater than zero, if it is, continue, otherwise escape. 

```javascript
if(_file.files.length === 0){
	return;
}
```

Now that's done, we can assume that there is a file selected, we're going to assume that there's one file, remember that arrays start with index of 0. 


###### Dynamically create the form data to send

To do this, we need to use something called `FormData`, and append the data to it with a name. When that's done, we can simply send this `FormData` up in the request that we generate in step 3. When using the `append` method, the first parameter is the same as the `name` attribute on an input, and the second parameter is the `value`. 

Here, we set the value to simply be the first file that's been selected in the files input.

```javascript
var data = new FormData();
data.append('SelectedFile', _file.files[0]);
```

We will use this later, when we send the data to the server.

###### Create an XMLHttpRequest to the upload script with event listeners

This part is pretty basic, we'll create a new `XMLHttpRequest`, and set some settings. Firstly we'll change the `onreadystatechange` parameter so that it is a function we define to check if the request is complete. This function will check the `readyState` every time it changes, and ensure that it is the value that we want - in this case it's 4,  meaning complete.

Secondly, we'll attach a `progress` event listener to the `upload` property. This is where the upload progress can be checked and used to update the progress bar.

```javascript
var request = new XMLHttpRequest();
request.onreadystatechange = function(){
	if(request.readyState == 4){
		try {
			var resp = JSON.parse(request.response);
		} catch (e){
			var resp = {
				status: 'error',
				data: 'Unknown error occurred: [' + request.responseText + ']'
			};
		}
		console.log(resp.status + ': ' + resp.data);
	}
};
```

When the request has completed successfully, we're simply checking that we've got valid JSON back using `try ... catch`, if it's not, we're going to create our own object so that the subsequent code can continue without any issues. How the response is handled can be changed, but currently it's just being outputted to the console.

Now let's handle the progress bar: 

```javascript
request.upload.addEventListener('progress', function(e){
	_progress.style.width = Math.ceil(e.loaded/e.total) * 100 + '%';
}, false);
```

Here is where there is a slight amount of complexity, we're attaching an event, and that event has a few properties associated with it, two of which we're interested in, `loaded` and `total`. `loaded` is a value of how much data has been sent to the server, and the `total` value is the overall amount of data to be sent, therefore we can use these two to work out a percentage, and set the progress bar to that width.

*Note:* There isn't anything fancy going on here, but there's nothing to stop the progress bar from being animated using some CSS animations or similar.

###### Send the upload

Now we can send the request, we're going to be doing this through `POST` to a file called `upload.php`, and using the `send()` method, with `data` as the parameter, we can send the correct data: 

```javascript
request.open('POST', 'upload.php');
request.send(data);
```

That should be all of the JavaScript completed giving you the following: 

```javascript
var _submit = document.getElementById('_submit'), 
_file = document.getElementById('_file'), 
_progress = document.getElementById('_progress');

var upload = function(){

	if(_file.files.length === 0){
	    return;
	}

	var data = new FormData();
	data.append('SelectedFile', _file.files[0]);

	var request = new XMLHttpRequest();
	request.onreadystatechange = function(){
	    if(request.readyState == 4){
	        try {
	            var resp = JSON.parse(request.response);
	        } catch (e){
	            var resp = {
	                status: 'error',
	                data: 'Unknown error occurred: [' + request.responseText + ']'
	            };
	        }
	        console.log(resp.status + ': ' + resp.data);
	    }
	};

	request.upload.addEventListener('progress', function(e){
	    _progress.style.width = Math.ceil(e.loaded/e.total) * 100 + '%';
	}, false);

	request.open('POST', 'upload.php');
	request.send(data);
}

_submit.addEventListener('click', upload);
```

Now on to the PHP...

##### PHP

This is the code that we're using, you'll notice some differences, mostly that all output is being sent as JSON using the top JSON method. This PHP is the **same** as that in the earlier tutorial, meaning it will only work with PNG images that are less than 500Kb. Also, the success message returns the path to the file that's been uploaded: 

```php
<?php
// Output JSON
function outputJSON($msg, $status = 'error'){
	header('Content-Type: application/json');
    die(json_encode(array(
        'data' => $msg,
        'status' => $status
    )));
}

// Check for errors
if($_FILES['SelectedFile']['error'] > 0){
    outputJSON('An error ocurred when uploading.');
}

if(!getimagesize($_FILES['SelectedFile']['tmp_name'])){
    outputJSON('Please ensure you are uploading an image.');
}

// Check filetype
if($_FILES['SelectedFile']['type'] != 'image/png'){
    outputJSON('Unsupported filetype uploaded.');
}

// Check filesize
if($_FILES['SelectedFile']['size'] > 500000){
    outputJSON('File uploaded exceeds maximum upload size.');
}

// Check if the file exists
if(file_exists('upload/' . $_FILES['SelectedFile']['name'])){
    outputJSON('File with that name already exists.');
}

// Upload file
if(!move_uploaded_file($_FILES['SelectedFile']['tmp_name'], 'upload/' . $_FILES['SelectedFile']['name'])){
    outputJSON('Error uploading file - check destination is writeable.');
}

// Success!
outputJSON('File uploaded successfully to "' . 'upload/' . $_FILES['SelectedFile']['name'] . '".', 'success');
```
If you put everything together you should get the file uploaded to where you're expecting it, with it returned as success within the console of the browser. 


#### Conclusion

Something that is really easy, but yet so effective, uploading images through AJAX can enhance your user experience with such ease. This can be expanded to add multiple uploaded files, by appending each one to the `FormData` that was created within the JavaScript. 

Just so that you're aware, if you're testing this locally, you might not see the progress bar's status change gradually, this is down to the speed of the upload on your local machine, I suggest using this on a server, with a relatively large PNG file.

<div class='post-footer-note'>
Note: This post was originally posted on Codular.com, but has since been migrated over.
</div>