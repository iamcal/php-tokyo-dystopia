TokyoDystopiaSimple
===================

The Simple interface allows you to index records as arrays of strings, then search for them
using a compound search expression syntax.

The class is a thin wrapper for the Simple API, which is documented here:
<a href="http://fallabs.com/tokyodystopia/spex.html#laputaapi">http://fallabs.com/tokyodystopia/spex.html#laputaapi</a>


Creating a database
-------------------

Creating a new database object is easy.

	$db = new TokyoDystopiaSimple();

The database is not linked to a file at this point, and cannot be used for reading or writing.
To do anything interesting, we'll have to point it at a database instance (or create one) first.


Opening and closing
-------------------

To work with a database file, you'll need to open it. Clsoe it when you're done.

	$ok = $db->open($path, $flags);
	$ok = $db->close();

The <code>$path</code> points at the <i>directory</i> you want the database to be created in.
Multiple files will reside in there, ubt you don't need to worry about those directly.

The following flags are valid and can be binary OR'd (<code>|</code>) together:

	TOKYO_DYSTOPIA_JDBOWRITER - open in write mode
	TOKYO_DYSTOPIA_JDBOREADER - open in read mode
	TOKYO_DYSTOPIA_JDBOCREAT - create a new db if it doesn't exist (write mode only)
	TOKYO_DYSTOPIA_JDBOTRUNC - create w new db even if it exists (write mode only)
	TOKYO_DYSTOPIA_JDBONOLCK - don't use file locking
	TOKYO_DYSTOPIA_JDBOLCKNB - use non-blocking locks

You <b>must</b> specify either read or write mode.
Both methods return true on success and false on failure.


Error handling
--------------

When something goes wrong, you can find out the error code and a human readable message.

	$code = $db->ecode();
	$msg = $db->errmsg();

The error codes are not defined as constants in PHP. You can find them in <code>tcutil.h</code> in the TokyoCabinet source.


Storing data
------------

To create or update a record in the index, we need to choose a numeric ID for it.
It has to be larger than 0 and (since this is PHP) smaller than 2147483647 (or maybe 9223372036854775807 on a 64bit system).

	$db->put($id, array("list", "of", "strings"));

You can retrieve any records you store by ID too.

	$strings = $db->get($id);

And also delete existing entries.

	$db->out($id);

Both the <code>put()</code> and <code>out()</code> methods return true on success and false on failure.
The <code>get()</code> methods returns an array on success and false on failure.


Searching
---------

Once you have some records in your database, searching for them is easy.

	$ids = $db->search($query);

The <code>search()</code> method returns an array of IDs to records that match the search query.
The query uses the standard TokyoDystopia compound syntax.

	A B      : searches for records including the two words.
	A && B   : searches for records including the two words.
	A || B   : searches for records including the one or both of the two words.
	"A B..." : searches for records including the phrase word which can include space characters.
	[[A*]]   : searches for records including words beginning with the token.
	[[*A]]   : searches for records including words ending with the token.
	[[*A*]]  : searches for records including words including the token.

On failure, false is returned.
Under the hood, this method calls <code>tcjdbsearch2()</code>.


Iterating over all records
--------------------------

It's possible to iterate over all records stored in a database.

	if (!$db->iterinit()){
		die("can't initialize iteration: ".$db->errmsg());
	}

	while ($id = $db->iternext()){

		# do something with record $id
	}

There is no way to distingish between <code>iternext()</code> failing and reaching the last record - 0 is returned in both cases


Moar!!!
-------

	$db->tune($expected_records, $expected_tokens, $index_file_size, $options);
	$db->setcache($max_token_cache_size, $max_cached_leaf_nodes);
	$db->setfwmmax($max_forward_matching_expansion);

	$db->sync();
	$db->optimize();
	$db->vanish();
	$db->copy($new_path);
	$path = $db->path();
	$num_records = $db->rnum();
	$total_size = $db->fsiz();
