# PHP-LevelDB: The PHP Binding for LevelDB
[![Build Status](https://secure.travis-ci.org/reeze/php-leveldb.png)](http://travis-ci.org/reeze/php-leveldb)

## Requirements
- PHP >= 5.2
- LevelDB

You could get leveldb from: <http://code.google.com/p/leveldb/>

	$ wget http://leveldb.googlecode.com/files/leveldb-1.5.0.tar.gz
	$ tar zxvf leveldb-1.5.0.tar.gz
	$ cd leveldb-1.5.0
	$ make

>**NOTE** LevelDB didn't have make install target in Makefile:
><http://code.google.com/p/leveldb/issues/detail?id=27>，
>If you want to install to a specific dir, you could:
>`make INSTALL_PATH=/Your/Path/`

## Installation

	$ git clone https://github.com/reeze/php-leveldb.git
	$ cd php-leveldb
	$ phpize
	$ ./configure --with-leveldb=/path/to/your/leveldb-1.*.*
	$ make
	$ make install

1. Install from PECL

	PHP-LevelDB havn't been hosted in PECL Yet

## Usage
Since PHP-LevelDB is a binding for LevelDB, most of the interface are the same as
LevelDB document: <http://leveldb.googlecode.com/git/doc/index.html>

### Open options
When open a leveldb database you could specify options to override default value:

````php
<?php
/* default open options */
$options = array(
	'create_if_missing' => true,	// if the specified database didn't exist will create a new one
	'error_if_exists'	=> false,	// if the opened database exsits will throw exception
	'paranoid_checks'	=> false,
	'block_cache_size'	=> 8 * (2 << 20),
	'write_buffer_size' => 4<<20,
	'block_size'		=> 4096,
	'max_open_files'	=> 1000,
	'block_restart_interval' => 16,
	'compression'		=> LEVELDB_SNAPPY_COMPRESSION,
	'comparator'		=> NULL,   // any callable parameter return 0, -1, 1
);
/* default readoptions */
$readoptions = array(
	'verify_check_sum'	=> false,
	'fill_cache'		=> true,
);

/* default write options */
$writeoptions = array(
	'sync' => false
);

$db = new LevelDB("/path/to/db", $options, $readoptions, $writeoptions);
````

>**NOTE** The readoptions and writeoptions will take effect when operate on
>db afterward, but you could override it by specify read/write options when
>accessing

### Using custom comparator
You could write your own comparator, the comparator can be anything callale in php
it the same as usort()'s compare function: <http://php.net/usort>, and the comparator
could be:

````php
<?php
int callback(string $a, string $b );
$db = new LevelDB("/path/to/db", array('comparator' => 'cmp'));
function cmp($a, $b)
{
	return strcmp($a, $b);
}
````

>**NOTE**
>If you create a database with custom comparator, you can only open it again
>with the same comparator.

### Basic operations: get(), put(), delete()
LevelDB is a key-value database, you could do those basic operations on it:

````php
<?php

$db = new LevelDB("/path/to/leveldb-test-db");

/*
 * Basic operate methods: set(), get(), delete()
 */
$db->put("Key", "Value");
$db->set("Key2", "Value2"); // set() is an alias of put()
$db->get("Key");
$db->delete("Key");
````

>**NOTE**
>Some key-value db use set instead of put to set value, so if like set(),
>you could use set() to save value.

### Write in a batch
If you want to do a sequence of update and want to make it atomically,
then writebatch will be your friend.

>The WriteBatch holds a sequence of edits to be made to the database, 
>and these edits within the batch are applied in order.
>
>Apart from its atomicity benefits, WriteBatch may also be used to speed up
>bulk updates by placing lots of individual mutations into the same batch.

````php
<?php

$db = new LevelDB("/path/to/leveldb-test-db");

$batch = new LevelDBWriteBatch();
$batch->put("key2", "batch value");
$batch->put("key3", "batch value");
$batch->set("key4", "a bounch of values"); // set() is an alias of put()
$batch->delete("some key");

// Write once
$db->write($batch);
````

### Iterate throught db

You can iterate through the whole database by iteration:

````php
<?php

$db = new LevelDB("/path/to/leveldb-test-db");
$it = new LevelDBIterator($db);

// Loop in iterator style
while($it->valid()) {
	var_dump($it->key() . " => " . $it->current() . "\n");
}

// Or loop in foreach
foreach($it as $key => $value) {
	echo "{$key} => {$value}\n";
}
````

if you want to iterate by reverse order, you could:

````php
<?php

$db = new LevelDB("/path/to/leveldb-test-db");
$it = new LevelDBIterator($db);

for($it->last(); $it->valid(); $it->prev()) {
	echo "{$key} => {$value}\n";
}

/*
 * And you could seek with: rewind(), next(), prev(), seek()
 */
````

>**NOTE** In LevelDB LevelDB::seek() will success even when the key didn't exists,
>it will seek to the latest key:
>`db-with-key('a', 'b', 'd', 'e');  $db->seek('c');` iterator will point to `key 'd'`

## Operations on database

### LevelDB::close()
Since leveldb can only accessed by a single proccess once, so you may want to
close it when you don't use it anymore.

````php
<?php

$db = new LevelDB("/path/to/leveldb-test-db");
$it = new LevelDBIterator($db);
$db->close();
$it->next();				// noop you can't do that, exception thrown
$db->set("key", "value");	// you can't do this either
````

after database closed, you can't do anything related to it;

### LevelDB::destroy()
If you want to destroy a database, you could delete the whole database by hand:
`rm -rf /path/to/db` or you could use `LevelDB::destroy('/path/to/db')`.

Be careful with this.

### LevelDB::repair()
If you can't open a database, neither been locked or other error, if it's corrupted,
you could use `LevelDB::repair('/path/to/db')` to repair it. it will try to recover
as much data as possible.

## Reference
More info could be found at:

- LevelDB project home: <http://code.google.com/p/leveldb/>
- LevelDB document: <http://leveldb.googlecode.com/git/doc/index.html>
- A LevelDB internals analysis in Chinese <http://dirlt.com/LevelDB.html> 推荐关注博主的博客

## License
PHP-LevelDB is licensed under PHP License
