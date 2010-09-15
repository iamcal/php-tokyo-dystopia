TokyoDystopiaSimple
===================

There are 4 different database types in TokyoDystopia, each with slightly different properties but all
with very similar APIs. Records are either a simple string, or an array of strings. Storage is either
full text and index, or pure index.

<table>
        <tr><th>Type</th><th>Records</th><th>Full text</th><th>Search type</th></tr>
        <tr><td>Core / Indexed</td><td align="center">Strings</td><td align="center">Yes</td><td align="center">Compound</td></tr>
        <tr><td>Q-Gram</td><td align="center">Strings</td><td align="center">No</td><td align="center">Single term with substring</td></tr>
        <tr><td>Simple / Tagged</td><td align="center">Arrays of Strings</td><td align="center">Yes</td><td align="center">Compound</td></tr>
        <tr><td>Word</td><td align="center">Arrays of Strings</td><td align="center">No</td><td align="center">Single term</td></tr>
</table>

This extension is a thin wrapper for the various APIs, which are all documented here:
<a href="http://fallabs.com/tokyodystopia/spex.html">http://fallabs.com/tokyodystopia/spex.html</a>


Creating a database
-------------------

Creating a new database object is easy.

	$db = new TokyoDystopiaCore();
	$db = new TokyoDystopiaQgram();
	$db = new TokyoDystopiaSimple();
	$db = new TokyoDystopiaWord();

The database is not linked to a file at this point, and cannot be used for reading or writing.
To do anything interesting, we'll have to point it at a database instance (or create one) first.


Opening and closing
-------------------

To work with a database file, you'll need to open it. Close it when you're done.

	$ok = $db->open($path, $flags);
	$ok = $db->close();

The <code>$path</code> points at the <i>directory</i> you want the database to be created in.
Multiple files will reside in there, ubt you don't need to worry about those directly.

The following flags are valid and can be binary OR'd (<code>|</code>) together:

	TOKYO_DYSTOPIA_*DBOWRITER - open in write mode
	TOKYO_DYSTOPIA_*DBOREADER - open in read mode
	TOKYO_DYSTOPIA_*DBOCREAT - create a new db if it doesn't exist (write mode only)
	TOKYO_DYSTOPIA_*DBOTRUNC - create w new db even if it exists (write mode only)
	TOKYO_DYSTOPIA_*DBONOLCK - don't use file locking
	TOKYO_DYSTOPIA_*DBOLCKNB - use non-blocking locks

You need to replace the <code>*</code> in the constant with either <code>I</code>, <code>Q</code>, 
<code>J</code> or <code>W</code> for Core, Q-Gram, Simple and Word databases respectively.

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
The type of data stored depends on the type of database you've created.

	# Core & Q-Gram 
	$db->put($id, "my string");

	# Simple & Word
	$db->put($id, array("list", "of", "strings"));

You can retrieve any records you store by ID too, for Core or Simple databases.

	# Core
	$string = $db->get($id);

	# Simple
	$strings = $db->get($id);

And also delete existing entries.

	$db->out($id);

Both the <code>put()</code> and <code>out()</code> methods return true on success and false on failure.
The <code>get()</code> methods returns a string or an array on success and false on failure.


Searching
---------

Once you have some records in your database, searching for them is easy.

	$ids = $db->search($query);

The <code>search()</code> method returns an array of IDs to records that match the search query.

When using the Core or Simple database, the query uses the standard TokyoDystopia compound syntax.

	A B      : searches for records including the two words.
	A && B   : searches for records including the two words.
	A || B   : searches for records including the one or both of the two words.
	"A B..." : searches for records including the phrase word which can include space characters.
	[[A*]]   : searches for records including words beginning with the token.
	[[*A]]   : searches for records including words ending with the token.
	[[*A*]]  : searches for records including words including the token.

On failure, false is returned.
Under the hood, this method calls the <code>*search2()</code> function.

The the Word database, the query is simply a single word to match.
The Q-Gram database has slightly different syntax:

	$ids = $db->search($query, $mode);

As with the Word database, the query is a single word, but you must also specify a search mode.

	TOKYO_DYSTOPIA_QDBSSUBSTR	Substring match
	TOKYO_DYSTOPIA_QDBSPREFIX	Prefix match
	TOKYO_DYSTOPIA_QDBSSUFFIX	Suffix match
	TOKYO_DYSTOPIA_QDBSFULL		Full match


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
