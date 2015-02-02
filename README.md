flow.js php server [![Build Status](https://travis-ci.org/flowjs/flow-php-server.png?branch=master)](https://travis-ci.org/flowjs/flow-php-server) [![Coverage Status](https://coveralls.io/repos/flowjs/flow-php-server/badge.png?branch=master)](https://coveralls.io/r/flowjs/flow-php-server?branch=master)
=======================

PHP library for handling chunk uploads. Library contains helper methods for:
 * Testing if uploaded file chunk exists.
 * Validating file chunk
 * Creating separate chunks folder
 * Validating uploaded chunks
 * Merging all chunks to a single file

This library is compatible with HTML5 file upload library: https://github.com/flowjs/flow.js

Installation
--------------
For easy installation you need to have Composer installed, if you don't please read installation document for Composer at https://getcomposer.org.

Clone this repository first:
```
git clone https://github.com/ImanMh/flow-php-server.git
```
then cd into the cloned directory:
```
cd flow-php-server
```
use composer to download dependencies and autoload PHP classes:
```
composer install
```
create a new php file for uploading files to it:
```
touch upload.php
```
Edit upload.php and add these lines:
```php
namespace Flow;
require_once './flow-php-server/vendor/autoload.php';
```
Note that you could also use ```use``` keyword to add each class inidividually into our current namespace. At this point you are ready to use flow. for more information about how to use it read Basic Usage section. 

Basic Usage
--------------
```php
if (\Flow\Basic::save('./final_file_destination', './chunks_temp_folder')) {
  // file saved successfully and can be accessed at './final_file_destination'
} else {
  // This is not a final chunk or request is invalid, continue to upload.
}
```
Make sure that `./chunks_temp_folder` path exists. All chunks will be save in this temporary folder.

If you are stuck with this example, please read this issue: [How to use the flow-php-server](https://github.com/flowjs/flow-php-server/issues/3#issuecomment-46979467)

Advanced Usage
--------------

```php
$config = new \Flow\Config();
$config->setTempDir('./chunks_temp_folder');
$file = new \Flow\File($config);

if ($_SERVER['REQUEST_METHOD'] === 'GET') {
    if ($file->checkChunk()) {
        header("HTTP/1.1 200 Ok");
    } else {
        header("HTTP/1.1 204 No Content");
        return ;
    }
} else {
  if ($file->validateChunk()) {
      $file->saveChunk();
  } else {
      // error, invalid chunk upload request, retry
      header("HTTP/1.1 400 Bad Request");
      return ;
  }
}
if ($file->validateFile() && $file->save('./final_file_name')) {
    // File upload was completed
} else {
    // This is not a final chunk, continue to upload
}
```

Delete unfinished files
-----------------------

For this you should setup cron, which would check each chunk upload time.
If chunk is uploaded long time ago, then chunk should be deleted.

Helper method for checking this:
```php
\Flow\Uploader::pruneChunks('./chunks_folder');
```

Cron task can be avoided by using random function execution.
```php
if (1 == mt_rand(1, 100)) {
    \Flow\Uploader::pruneChunks('./chunks_folder');
}
```

Contribution
------------

Your participation in development is very welcome!

To ensure consistency throughout the source code, keep these rules in mind as you are working:
 * All features or bug fixes must be tested by one or more specs.
 * Your code should follow [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) coding style guide
