# NAME

IO::All - IO::All of it to Graham and Damian!

# SYNOPSIS

    use IO::All;                                # Let the madness begin...

    # Some of the many ways to read a whole file into a scalar
    io('file.txt') > $contents;                 # Overloaded "arrow"
    $contents < io 'file.txt';                  # Flipped but same operation
    $io = io 'file.txt';                        # Create a new IO::All object
    $contents = $$io;                           # Overloaded scalar dereference
    $contents = $io->all;                       # A method to read everything
    $contents = $io->slurp;                     # Another method for that
    $contents = join '', $io->getlines;         # Join the separate lines
    $contents = join '', map "$_\n", @$io;      # Same. Overloaded array deref
    $io->tie;                                   # Tie the object as a handle
    $contents = join '', <$io>;                 # And use it in builtins
    # and the list goes on ...

    # Other file operations:
    @lines = io('file.txt')->slurp;             # List context slurp
    $content > io('file.txt');                  # Print to a file
    io('file.txt')->print($content, $more);     # (ditto)
    $content >> io('file.txt');                 # Append to a file
    io('file.txt')->append($content);           # (ditto)
    $content << $io;                            # Append to a string
    io('copy.txt') < io('file.txt');            $ Copy a file
    io('file.txt') > io('copy.txt');            # Invokes File::Copy
    io('more.txt') >> io('all.txt');            # Add on to a file

    # UTF-8 Support
    $contents = io('file.txt')->utf8->all;      # Turn on utf8
    use IO::All -utf8;                          # Turn on utf8 for all io
    $contents = io('file.txt')->all;            #   by default in this package.

    # General Encoding Support
    $contents = io('file.txt')->encoding('big5')->all;
    use IO::All -encoding => 'big5';            # Turn on big5 for all io
    $contents = io('file.txt')->all;            #   by default in this package.

    # Print the path name of a file:
    print $io->name;                            # The direct method
    print "$io";                                # Object stringifies to name
    print $io;                                  # Quotes not needed here
    print $io->filename;                        # The file portion only

    # Read all the files/directories in a directory:
    $io = io('my/directory/');                  # Create new directory object
    @contents = $io->all;                       # Get all contents of dir
    @contents = @$io;                           # Directory as an array
    @contents = values %$io;                    # Directory as a hash
    push @contents, $subdir                     # One at a time
      while $subdir = $io->next;

    # Print the name and file type for all the contents above:
    print "$_ is a " . $_->type . "\n"          # Each element of @contents
      for @contents;                            # is an IO::All object!!

    # Print first line of each file:
    print $_->getline                           # getline gets one line
      for io('dir')->all_files;                 # Files only

    # Print names of all files/dirs three directories deep:
    print "$_\n" for $io->all(3);               # Pass in the depth. Default=1

    # Print names of all files/dirs recursively:
    print "$_\n" for $io->all(0);               # Zero means all the way down
    print "$_\n" for $io->All;                  # Capitalized shortcut
    print "$_\n" for $io->deep->all;            # Another way

    # There are some special file names:
    print io('-');                              # Print STDIN to STDOUT
    io('-') > io('-');                          # Do it again
    io('-') < io('-');                          # Same. Context sensitive.
    "Bad puppy" > io('=');                      # Message to STDERR
    $string_file = io('$');                     # Create IO::String Object
    $temp_file = io('?');                       # Create a temporary file

    # Socket operations:
    $server = io('localhost:5555')->fork;       # Create a daemon socket
    $connection = $server->accept;              # Get a connection socket
    $input < $connection;                       # Get some data from it
    "Thank you!" > $connection;                 # Thank the caller
    $connection->close;                         # Hang up
    io(':6666')->accept->slurp > io->devnull;   # Take a complaint and file it
    

    # DBM database operations:
    $dbm = io 'my/database';                    # Create a database object
    print $dbm->{grocery_list};                 # Hash context makes it a DBM
    $dbm->{todo} = $new_list;                   # Write to database
    $dbm->dbm('GDBM_file');                     # Demand specific DBM
    io('mydb')->mldbm->{env} = \%ENV;           # MLDBM support

    # Tie::File support:
    $io = io 'file.txt';
    $io->[42] = 'Line Forty Three';             # Change a line
    print $io->[@$io / 2];                      # Print middle line
    @$io = reverse @$io;                        # Reverse lines in a file

    # Stat functions:
    printf "%s %s %s\n",                        # Print name, uid and size of 
      $_->name, $_->uid, $_->size               # contents of current directory
        for io('.')->all;
    print "$_\n" for sort                       # Use mtime method to sort all
      {$b->mtime <=> $a->mtime}                 # files under current directory
        io('.')->All_Files;                     # by recent modification time.

    # File::Spec support:
    $contents < io->catfile(qw(dir file.txt));  # Portable IO operation
    

    # Miscellaneous:
    @lines = io('file.txt')->chomp->slurp;      # Chomp as you slurp
    @chunks = 
      io('file.txt')->separator('xxx')->slurp;  # Use alternnate record sep
    $binary = io('file.bin')->binary->all;      # Read a binary file
    io('a-symlink')->readlink->slurp;           # Readlink returns an object
    print io('foo')->absolute->pathname;        # Print absolute path of foo

    # IO::All External Plugin Methods
    io("myfile") > io->("ftp://store.org");     # Upload a file using ftp
    $html < io->http("www.google.com");         # Grab a web page
    io('mailto:worst@enemy.net')->print($spam); # Email a "friend"

    # This is just the beginning, read on...

# DESCRIPTION

"Graham Barr for doing it all. Damian Conway for doing it all different."

IO::All combines all of the best Perl IO modules into a single nifty
object oriented interface to greatly simplify your everyday Perl IO
idioms. It exports a single function called `io`, which returns a new
IO::All object. And that object can do it all!

The IO::All object is a proxy for IO::File, IO::Dir, IO::Socket,
IO::String, Tie::File, File::Spec, File::Path and File::ReadBackwards;
as well as all the DBM and MLDBM modules. You can use most of the
methods found in these classes and in IO::Handle (which they inherit
from). IO::All adds dozens of other helpful idiomatic methods
including file stat and manipulation functions. 

IO::All is pluggable, and modules like IO::All::LWP and IO::All::Mailto
add even more functionality. Optionally, every IO::All object can be
tied to itself. This means that you can use most perl IO builtins on it:
readline, <>, getc, print, printf, syswrite, sysread, close.

The distinguishing magic of IO::All is that it will automatically open
(and close) files, directories, sockets and other IO things for you. You
never need to specify the mode ('<', '>>', etc), since it is determined
by the usage context. That means you can replace this:

    open STUFF, '<', './mystuff'
      or die "Can't open './mystuff' for input:\n$!";
    local $/;
    my $stuff = <STUFF>;
    close STUFF;

with this:

    my $stuff < io"./mystuff";

And that is a __good thing__!

# USAGE

Normally just say:

    use IO::All;

and IO::All will export a single function called `io`, which contructs all IO
objects.

You can also pass global flags like this:

    use IO::All -strict -encoding => 'big5', -foobar;

Which automatically makes those method calls on every new IO object. In other
words this:

    my $io = io('lalala.txt');

becomes this:

    my $io = io('lalala.txt')->strict->encoding('big5')->foobar;

# METHOD ROLE CALL

Here is an alphabetical list of all the public methods that you can call
on an IO::All object.

`abs2rel`, `absolute`, `accept`, `All`, `all`, `All_Dirs`,
`all_dirs`, `All_Files`, `all_files`, `All_Links`, `all_links`,
`append`, `appendf`, `appendln`, `assert`, `atime`, `autoclose`,
`autoflush`, `backwards`, `bcc`, `binary`, `binmode`, `blksize`,
`blocks`, `block_size`, `buffer`, `canonpath`, `case_tolerant`,
`catdir`, `catfile`, `catpath`, `cc`, `chdir`, `chomp`, `clear`,
`close`, `confess`, `content`, `ctime`, `curdir`, `dbm`, `deep`,
`device`, `device_id`, `devnull`, `dir`, `domain`, `empty`,
`encoding`, `eof`, `errors`, `file`, `filename`, `fileno`,
`filepath`, `filter`, `fork`, `from`, `ftp`, `get`, `getc`,
`getline`, `getlines`, `gid`, `handle`, `head`, `http`, `https`,
`inode`, `io_handle`, `is_absolute`, `is_dir`, `is_dbm`,
`is_executable`, `is_file`, `is_link`, `is_mldbm`, `is_open`,
`is_pipe`, `is_readable`, `is_socket`, `is_stdio`, `is_string`,
`is_temp`, `is_writable`, `join`, `length`, `link`, `lock`,
`mailer`, `mailto`, `mkdir`, `mkpath`, `mldbm`, `mode`, `modes`,
`mtime`, `name`, `new`, `next`, `nlink`, `open`, `password`,
`path`, `pathname`, `perms`, `pipe`, `port`, `print`, `printf`,
`println`, `put`, `rdonly`, `rdwr`, `read`, `readdir`,
`readlink`, `recv`, `rel2abs`, `relative`, `rename`, `request`,
`response`, `rmdir`, `rmtree`, `rootdir`, `scalar`, `seek`,
`send`, `separator`, `shutdown`, `size`, `slurp`, `socket`,
`sort`, `splitdir`, `splitpath`, `stat`, `stdio`, `stderr`,
`stdin`, `stdout`, `strict`, `string`, `string_ref`, `subject`,
`sysread`, `syswrite`, `tail`, `tell`, `temp`, `tie`, `tmpdir`,
`to`, `touch`, `truncate`, `type`, `user`, `uid`, `unlink`,
`unlock`, `updir`, `uri`, `utf8`, `utime` and `write`.

Each method is documented further below.

# OPERATOR OVERLOADING

IO::All objects overload a small set of Perl operators to great effect.
The overloads are limited to <, <<, >, >>, dereferencing operations, and
stringification.

Even though relatively few operations are overloaded, there is actually
a huge matrix of possibilities for magic. That's because the overloading
is sensitive to the types, position and context of the arguments, and an
IO::All object can be one of many types.

The most important overload to grok is stringification. IO::All objects
stringify to their file or directory name. Here we print the contents of
the current directory:

    perl -MIO::All -le 'print for io(".")->all'

is the same as:

    perl -MIO::All -le 'print $_->name for io(".")->all'

Stringification is important because it allows IO::All operations to return
objects when they might otherwise return file names. Then the recipient can
use the result either as an object or a string.

'>' and '<' move data between objects in the direction pointed to by the
operator.

    $content1 < io('file1');
    $content1 > io('file2');
    io('file2') > $content3;
    io('file3') < $content3;
    io('file3') > io('file4');
    io('file5') < io('file4');

'>>' and '<<' do the same thing except the recipent string or file is
appended to.

An IO::All file used as an array reference becomes tied using Tie::File:

    $file = io"file";
    # Print last line of file
    print $file->[-1];
    # Insert new line in middle of file
    $file->[$#$file / 2] = 'New line';

An IO::All file used as a hash reference becomes tied to a DBM class:

    io('mydbm')->{ingy} = 'YAML';

An IO::All directory used as an array reference, will expose each file or
subdirectory as an element of the array.

    print "$_\n" for @{io 'dir'};

IO::All directories used as hash references have file names as keys, and
IO::All objects as values:

    print io('dir')->{'foo.txt'}->slurp;

Files used as scalar references get slurped:

    print ${io('dir')->{'foo.txt'}};

Not all combinations of operations and object types are supported. Some
just haven't been added yet, and some just don't make sense. If you use
an invalid combination, an error will be thrown.

# COOKBOOK

This section describes some various things that you can easily cook up
with IO::All.

## File Locking

IO::All makes it very easy to lock files. Just use the `lock` method. Here's a
standalone program that demonstrates locking for both write and read:

    use IO::All;
    my $io1 = io('myfile')->lock;
    $io1->println('line 1');

    fork or do {
        my $io2 = io('myfile')->lock;
        print $io2->slurp;
        exit;
    };

    sleep 1;
    $io1->println('line 2');
    $io1->println('line 3');
    $io1->unlock;

There are a lot of subtle things going on here. An exclusive lock is
issued for `$io1` on the first `println`. That's because the file
isn't actually opened until the first IO operation.

When the child process tries to read the file using `$io2`, there is
a shared lock put on it. Since `$io1` has the exclusive lock, the
slurp blocks.

The parent process sleeps just to make sure the child process gets a
chance. The parent needs to call `unlock` or `close` to release the
lock. If all goes well the child will print 3 lines.

## Round Robin

This simple example will read lines from a file forever. When the last
line is read, it will reopen the file and read the first one again.

    my $io = io'file1.txt';
    $io->autoclose(1);
    while (my $line = $io->getline || $io->getline) {
        print $line;
    }

## Reading Backwards

If you call the `backwards` method on an IO::All object, the
`getline` and `getlines` will work in reverse. They will read the
lines in the file from the end to the beginning.

    my @reversed;
    my $io = io('file1.txt');
    $io->backwards;
    while (my $line = $io->getline) {
        push @reversed, $line;
    }

or more simply:

    my @reversed = io('file1.txt')->backwards->getlines;

The `backwards` method returns the IO::All object so that you can
chain the calls.

NOTE: This operation requires that you have the File::ReadBackwards 
module installed.
    

## Client/Server Sockets

IO::All makes it really easy to write a forking socket server and a
client to talk to it.

In this example, a server will return 3 lines of text, to every client
that calls it. Here is the server code:

    use IO::All;

    my $socket = io(':12345')->fork->accept;
    $socket->print($_) while <DATA>;
    $socket->close;

    __DATA__
    On your mark,
    Get set,
    Go!

Here is the client code:

    use IO::All;

    my $io = io('localhost:12345');
    print while $_ = $io->getline;

You can run the server once, and then run the client repeatedly (in
another terminal window). It should print the 3 data lines each time.

Note that it is important to close the socket if the server is forking,
or else the socket won't go out of scope and close.

## A Tiny Web Server

Here is how you could write a simplistic web server that works with static and
dynamic pages:

    perl -MIO::All -e 'io(":8080")->fork->accept->(sub { $_[0] < io(-x $1 ? "./$1 |" : $1) if /^GET \/(.*) / })'

There is are a lot of subtle things going on here. First we accept a socket
and fork the server. Then we overload the new socket as a code ref. This code
ref takes one argument, another code ref, which is used as a callback. 

The callback is called once for every line read on the socket. The line
is put into `$_` and the socket itself is passed in to the callback.

Our callback is scanning the line in `$_` for an HTTP GET request. If one is
found it parses the file name into `$1`. Then we use `$1` to create an new
IO::All file object... with a twist. If the file is executable (`-x`), then
we create a piped command as our IO::All object. This somewhat approximates
CGI support.

Whatever the resulting object is, we direct the contents back at our socket
which is in `$_[0]`. Pretty simple, eh? 

## DBM Files

IO::All file objects used as a hash reference, treat the file as a DBM tied to
a hash. Here I write my DB record to STDERR:

    io("names.db")->{ingy} > io'=';

Since their are several DBM formats available in Perl, IO::All picks the first
one of these that is installed on your system:

    DB_File GDBM_File NDBM_File ODBM_File SDBM_File

You can override which DBM you want for each IO::All object:

    my @keys = keys %{io('mydbm')->dbm('SDBM_File')};

## File Subclassing

Subclassing is easy with IO::All. Just create a new module and use
IO::All as the base class, like this:

    package NewModule;
    use IO::All -base;

You need to do it this way so that IO::All will export the `io` function.
Here is a simple recipe for subclassing:

IO::Dumper inherits everything from IO::All and adds an extra method
called `dump`, which will dump a data structure to the file we
specify in the `io` function. Since it needs Data::Dumper to do the
dumping, we override the `open` method to `require Data::Dumper` and
then pass control to the real `open`.

First the code using the module:

    use IO::Dumper;
    

    io('./mydump')->dump($hash);

And next the IO::Dumper module itself:

    package IO::Dumper;
    use IO::All -base;
    use Data::Dumper;
    

    sub dump {
        my $self = shift;
        Dumper(@_) > $self;
    }
    

    1;

## Inline Subclassing

This recipe does the same thing as the previous one, but without needing
to write a separate module. The only real difference is the first line.
Since you don't "use" IO::Dumper, you need to still call its `import`
method manually.

    IO::Dumper->import;
    io('./mydump')->dump($hash);
    

    package IO::Dumper;
    use IO::All -base;
    use Data::Dumper;
    

    sub dump {
        my $self = shift;
        Dumper(@_) > $self;
    }

# THE IO::All METHODS

This section gives a full description of all of the methods that you can
call on IO::All objects. The methods have been grouped into subsections
based on object construction, option settings, configuration, action
methods and support for specific modules.

## Object Construction and Initialization Methods

- * new

There are three ways to create a new IO::All object. The first is with
the special function `io` which really just calls `IO::All->new`.
The second is by calling `new` as a class method. The third is calling
`new` as an object instance method. In this final case, the new objects
attributes are copied from the instance object.

    io(file-descriptor);
    IO::All->new(file-descriptor);
    $io->new(file-descriptor);
            

All three forms take a single argument, a file descriptor. A file
descriptor can be any of the following:

    - A file name
    - A file handle
    - A directory name
    - A directory handle
    - A typeglob reference
    - A piped shell command. eq '| ls -al'
    - A socket domain/port.  eg 'perl.com:5678'
    - '-' means STDIN or STDOUT (depending on usage)
    - '=' means STDERR
    - '$' means an IO::String object
    - '?' means a temporary file
    - A URI including: http, https, ftp and mailto
    - An IO::All object

If you provide an IO::All object, you will simply get that _same
object_ returned from the constructor.

If no file descriptor is provided, an object will still be created, but
it must be defined by one of the following methods before it can be used
for I/O:

- * file

    io->file(file-name);

Using the `file` method sets the type of the object to _file_ and sets
the pathname of the file if provided.

It might be important to use this method if you had a file whose name
was `'-'`, or if the name might otherwise be confused with a
directory or a socket. In this case, either of these statements would
work the same:

    my $file = io('-')->file;
    my $file = io->file('-');

- * dir

    io->file(dir-name);

Make the object be of type _directory_.

- * socket

    io->file(domain:port);

Make the object be of type _socket_.

- * link

    io->file(link-name);

Make the object be of type _link_.

- * pipe

    io->file(link-name);

Make the object be of type _pipe_. The following two statements are
equivalent:

    my $io = io('ls -l |');
    my $io = io('ls -l')->pipe;
    my $io = io->pipe('ls -l');

- * dbm

This method takes the names of zero or more DBM modules. The first one
that is available is used to process the dbm file.

    io('mydbm')->dbm('NDBM_File', 'SDBM_File')->{author} = 'ingy';

If no module names are provided, the first available of the
following is used:

    DB_File GDBM_File NDBM_File ODBM_File SDBM_File

- * mldbm

Similar to the `dbm` method, except create a Multi Level DBM object
using the MLDBM module.

This method takes the names of zero or more DBM modules and an optional
serialization module. The first DBM module that is available is used to
process the MLDBM file. The serialization module can be Data::Dumper,
Storable or FreezeThaw.

    io('mymldbm')->mldbm('GDBM_File', 'Storable')->{author} = 
      {nickname => 'ingy'};

- * string

Make the object be a IO::String object. These are equivalent:

    my $io = io('$');
    my $io = io->string;

- * temp

Make the object represent a temporary file. It will automatically be
open for both read and write.

- * stdio

Make the object represent either STDIN or STDOUT depending on how it is
used subsequently. These are equivalent:

    my $io = io('-');
    my $io = io->stdin;

- * stdin

Make the object represent STDIN.

- * stdout

Make the object represent STDOUT.

- * stderr

Make the object represent STDERR.

- * handle

    io->handle(io-handle);

Forces the object to be created from an pre-existing IO handle. You can
chain calls together to indicate the type of handle:

    my $file_object = io->file->handle($file_handle);
    my $dir_object = io->dir->handle($dir_handle);

- * http

Make the object represent an http uri. Requires IO-All-LWP.

- * https

Make the object represent an https uri. Requires IO-All-LWP.

- * ftp

Make the object represent a ftp uri. Requires IO-All-LWP.

- * mailto

Make the object represent a mailto uri. Requires IO-All-Mailto.

If you need to use the same options to create a lot of objects, and
don't want to duplicate the code, just create a dummy object with the
options you want, and use that object to spawn other objects.

    my $lt = io->lock->tie;
    ...
    my $io1 = $lt->new('file1');
    my $io2 = $lt->new('file2');

Since the new method copies attributes from the calling object, both
`$io1` and `$io2` will be locked and tied.

## Option Setting Methods

The following methods don't do any actual I/O, but they specify options
about how the I/O should be done.

Each option can take a single argument of 0 or 1. If no argument is
given, the value 1 is assumed. Passing 0 turns the option off.

All of these options return the object reference that was used to
invoke them. This is so that the option methods can be chained
together. For example:

    my $io = io('path/file')->tie->assert->chomp->lock;

- * absolute

Indicates that the `pathname` for the object should be made absolute.

- * assert

This method ensures that the path for a file or directory actually exists
before the file is open. If the path does not exist, it is created.

- * autoclose

By default, IO::All will close an object opened for input when EOF is
reached. By closing the handle early, one can immediately do other
operations on the object without first having to close it.

This option is on by default, so if you don't want this behaviour, say
so like this:

    $io->autoclose(0);

The object will then be closed when `$io` goes out of scope, or you
manually call `$io->close`.

- * autoflush

Proxy for IO::Handle::autoflush

- * backwards

Sets the object to 'backwards' mode. All subsequent `getline`
operations will read backwards from the end of the file.

Requires the File::ReadBackwards CPAN module.

- * binary

Indicates the file has binary content and should be opened with
`binmode`.

- * chdir

chdir() to the pathname of a directory object. When object goes out of
scope, chdir back to starting directory.

- * chomp

Indicates that all operations that read lines should chomp the lines. If
the `separator` method has been called, chomp will remove that value
from the end of each record.

- * confess

Errors should be reported with the very detailed Carp::confess function.

- * deep

Indicates that calls to the `all` family of methods should search
directories as deep as possible.

- * fork

Indicates that the process should automatically be forked inside the
`accept` socket method.

- * lock

Indicate that operations on an object should be locked using flock.

- * rdonly

This option indicates that certain operations like DBM and Tie::File
access should be done in read-only mode.

- * rdwr

This option indicates that DBM and MLDBM files should be opened in read-
write mode.

- * relative

Indicates that the `pathname` for the object should be made relative.

- * sort

Indicates whether objects returned from one of the `all` methods will
be in sorted order by name. True by default.

- * strict

Check the return codes of every single system call. To turn this on for all
calls in your module, use:

    use IO::All -strict;

- * tie

Indicate that the object should be tied to itself, thus allowing it to
be used as a filehandle in any of Perl's builtin IO operations.

    my $io = io('foo')->tie;
    @lines = <$io>;

- * utf8

Indicates that IO should be done using utf8 encoding. Calls binmode with
`:utf8` layer.

## Configuration Methods

The following methods don't do any actual I/O, but they set specific
values to configure the IO::All object.

If these methods are passed no argument, they will return their
current value. If arguments are passed they will be used to set the
current value, and the object reference will be returned for potential
method chaining.

- * bcc

Set the Bcc field for a mailto object.

- * binmode

Proxy for binmode. Requires a layer to be passed. Use `binary` for
plain binary mode.

- * block_size

The default length to be used for `read` and `sysread` calls.
Defaults to 1024.

- * buffer

Returns a reference to the internal buffer, which is a scalar. You can
use this method to set the buffer to a scalar of your choice. (You can
just pass in the scalar, rather than a reference to it.)

This is the buffer that `read` and `write` will use by default.

You can easily have IO::All objects use the same buffer:

    my $input = io('abc');
    my $output = io('xyz');
    my $buffer;
    $output->buffer($input->buffer($buffer));
    $output->write while $input->read;

- * cc

Set the Cc field for a mailto object.

- * content

Get or set the content for an LWP operation manually.

- * domain

Set the domain name or ip address that a socket should use.

- * encoding

Set the encoding to be used for the PerlIO layer.

- * errors

Use this to set a subroutine reference that gets called when an internal
error is thrown.

- * filter

Use this to set a subroutine reference that will be used to grep
which objects get returned on a call to one of the `all` methods.
For example:

    my @odd = io->curdir->filter(sub {$_->size % 2})->All_Files;

`@odd` will contain all the files under the current directory whose
size is an odd number of bytes.

- * from

Indicate the sender for a mailto object.

- * mailer

Set the mailer program for a mailto transaction. Defaults to 'sendmail'.

- * mode

Set the mode for which the file should be opened. Examples:

    $io->mode('>>')->open;
    $io->mode(O_RDONLY);

    my $log_appender = io->file('/var/log/my-application.log')
                         ->mode('>>')->open();

    $log_appender->print("Stardate 5987.6: Mission accomplished.");

- * name

Set or get the name of the file or directory represented by the IO::All
object.

- * password

Set the password for an LWP transaction.

- * perms

Sets the permissions to be used if the file/directory needs to be created.

- * port

Set the port number that a socket should use.

- * request

Manually specify the request object for an LWP transaction.

- * response

Returns the resulting reponse object from an LWP transaction.

- * separator

Sets the record (line) separator to whatever value you pass it. Default
is \n. Affects the chomp setting too.

- * string_ref

Proxy for IO::String::string_ref

Returns a reference to the internal string that is acting like a file.

- * subject

Set the subject for a mailto transaction.

- * to

Set the recipient address for a mailto request.

- * uri

Direct access to the URI used in LWP transactions.

- * user

Set the user name for an LWP transaction.

## IO Action Methods

These are the methods that actually perform I/O operations on an IO::All
object. The stat methods and the File::Spec methods are documented in
separate sections below.

- * accept

For sockets. Opens a server socket (LISTEN => 1, REUSE => 1). Returns an
IO::All socket object that you are listening on.

If the `fork` method was called on the object, the process will
automatically be forked for every connection.

- * all

Read all contents into a single string.

    compare(io('file1')->all, io('file2')->all);

- * all (For directories)

Returns a list of IO::All objects for all files and subdirectories in a
directory. 

'.' and '..' are excluded.

Takes an optional argument telling how many directories deep to search. The
default is 1. Zero (0) means search as deep as possible.

The filter method can be used to limit the results.

The items returned are sorted by name unless `->sort(0)` is used.

- * All

Same as `all(0)`.

- * all_dirs

Same as `all`, but only return directories.

- * All_Dirs

Same as `all_dirs(0)`.

- * all_files

Same as `all`, but only return files.

- * All_Files

Same as `all_files(0)`.

- * all_links

Same as `all`, but only return links.

- * All_Links

Same as `all_links(0)`.

- * append

Same as print, but sets the file mode to '>>'.

- * appendf

Same as printf, but sets the file mode to '>>'.

- * appendln

Same as println, but sets the file mode to '>>'.

- * clear

Clear the internal buffer. This method is called by `write` after it
writes the buffer. Returns the object reference for chaining.

- * close

Close will basically unopen the object, which has different meanings for
different objects. For files and directories it will close and release
the handle. For sockets it calls shutdown. For tied things it unties
them, and it unlocks locked things.

- * empty

Returns true if a file exists but has no size, or if a directory exists but
has no contents.

- * eof

Proxy for IO::Handle::eof

- * exists

Returns whether or not the file or directory exists.

- * filename

Return the name portion of the file path in the object. For example:

    io('my/path/file.txt')->filename;

would return `file.txt`.

- * fileno

Proxy for IO::Handle::fileno

- * filepath

Return the path portion of the file path in the object. For example:

    io('my/path/file.txt')->filename;

would return `my/path`.

- * get

Perform an LWP GET request manually.

- * getc

Proxy for IO::Handle::getc

- * getline

Calls IO::File::getline. You can pass in an optional record separator.

- * getlines

Calls IO::File::getlines. You can pass in an optional record separator.

- * head

Return the first 10 lines of a file. Takes an optional argument which is the
number of lines to return. Works as expected in list and scalar context. Is
subject to the current line separator.

- * io_handle

Direct access to the actual IO::Handle object being used on an opened
IO::All object.

- * is_dir

Returns boolean telling whether or not the IO::All object represents
a directory.

- * is_executable

Returns true if file or directory is executable.

- * is_dbm

Returns boolean telling whether or not the IO::All object
represents a dbm file.

- * is_file

Returns boolean telling whether or not the IO::All object
represents a file.

- * is_link

Returns boolean telling whether or not the IO::All object represents
a symlink.

- * is_mldbm

Returns boolean telling whether or not the IO::All object
represents a mldbm file.

- * is_open

Indicates whether the IO::All is currently open for input/output.

- * is_pipe

Returns boolean telling whether or not the IO::All object represents
a pipe operation.

- * is_readable

Returns true if file or directory is readable.

- * is_socket

Returns boolean telling whether or not the IO::All object represents
a socket.

- * is_stdio

Returns boolean telling whether or not the IO::All object represents
a STDIO file handle.

- * is_string

Returns boolean telling whether or not the IO::All object represents
an IO::String object.

- * is_temp

Returns boolean telling whether or not the IO::All object represents
a temporary file.

- * is_writable

Returns true if file or directory is writable.  Can also be spelled as
`is_writeable`.

- * length

Return the length of the internal buffer.

- * mkdir

Create the directory represented by the object.

- * mkpath

Create the directory represented by the object, when the path contains
more than one directory that doesn't exist. Proxy for File::Path::mkpath.

- * next

For a directory, this will return a new IO::All object for each file
or subdirectory in the directory. Return undef on EOD.

- * open

Open the IO::All object. Takes two optional arguments `mode` and
`perms`, which can also be set ahead of time using the `mode` and
`perms` methods.

NOTE: Normally you won't need to call open (or mode/perms), since this
happens automatically for most operations.

- * pathname

Return the absolute or relative pathname for a file or directory, depending on
whether object is in `absolute` or `relative` mode.

- * print

Proxy for IO::Handle::print

- * printf

Proxy for IO::Handle::printf

- * println

Same as print, but adds newline to each argument unless it already
ends with one.

- * put

Perform an LWP PUT request manually.

- * read

This method varies depending on its context. Read carefully (no pun
intended).

For a file, this will proxy IO::File::read. This means you must pass
it a buffer, a length to read, and optionally a buffer offset for where
to put the data that is read. The function returns the length actually
read (which is zero at EOF).

If you don't pass any arguments for a file, IO::All will use its own
internal buffer, a default length, and the offset will always point at
the end of the buffer. The buffer can be accessed with the `buffer`
method. The length can be set with the `block_size` method. The default
length is 1024 bytes. The `clear` method can be called to clear
the buffer.

For a directory, this will proxy IO::Dir::read.

- * readdir

Similar to the Perl `readdir` builtin. In scalar context, return the next
directory entry (ie file or directory name), or undef on end of directory. In
list context, return all directory entries.

Note that `readdir` does not return the special `.` and `..` entries.

- * readline

Same as `getline`.

- * readlink

Calls Perl's readlink function on the link represented by the object.
Instead of returning the file path, it returns a new IO::All object
using the file path.

- * recv

Proxy for IO::Socket::recv

- * rename

    my $new = $io->rename('new-name');

Calls Perl's rename function and returns an IO::All object for the
renamed file. Returns false if the rename failed.

- * rewind

Proxy for IO::Dir::rewind

- * rmdir

Delete the directory represented by the IO::All object.

- * rmtree

Delete the directory represented by the IO::All object and all the files
and directories beneath it. Proxy for File::Path::rmtree.

- * scalar

Deprecated. Same as `all()`.

- * seek

Proxy for IO::Handle::seek. If you use seek on an unopened file, it will
be opened for both read and write.

- * send

Proxy for IO::Socket::send

- * shutdown

Proxy for IO::Socket::shutdown

- * slurp

Read all file content in one operation. Returns the file content
as a string. In list context returns every line in the file.

- * stat

Proxy for IO::Handle::stat

- * sysread

Proxy for IO::Handle::sysread

- * syswrite

Proxy for IO::Handle::syswrite

- * tail

Return the last 10 lines of a file. Takes an optional argument which is the
number of lines to return. Works as expected in list and scalar context. Is
subject to the current line separator.

- * tell

Proxy for IO::Handle::tell

- * throw

This is an internal method that gets called whenever there is an error.
It could be useful to override it in a subclass, to provide more control
in error handling.

- * touch

Update the atime and mtime values for a file or directory. Creates an empty
file if the file does not exist.

- * truncate

Proxy for IO::Handle::truncate

- * type

Returns a string indicated the type of io object. Possible values are:

    file
    dir
    link
    socket
    string
    pipe

Returns undef if type is not determinable.

- * unlink

Unlink (delete) the file represented by the IO::All object.

NOTE: You can unlink a file after it is open, and continue using it
until it is closed.

- * unlock

Release a lock from an object that used the `lock` method.

- * utime

Proxy for the utime Perl function.

- * write

Opposite of `read` for file operations only.

NOTE: When used with the automatic internal buffer, `write` will
clear the buffer after writing it.

## Stat Methods

This methods get individual values from a stat call on the file,
directory or handle represented by th IO::All object.

- * atime

Last access time in seconds since the epoch

- * blksize

Preferred block size for file system I/O

- * blocks

Actual number of blocks allocated

- * ctime

Inode change time in seconds since the epoch

- * device

Device number of filesystem

- * device_id

Device identifier for special files only

- * gid

Numeric group id of file's owner

- * inode

Inode number

- * modes

File mode - type and permissions

- * mtime

Last modify time in seconds since the epoch

- * nlink

Number of hard links to the file

- * size

Total size of file in bytes

- * uid

Numeric user id of file's owner

## File::Spec Methods

These methods are all adaptations from File::Spec. Each method
actually does call the matching File::Spec method, but the arguments
and return values differ slightly. Instead of being file and directory
__names__, they are IO::All __objects__. Since IO::All objects stringify
to their names, you can generally use the methods just like File::Spec.

- * abs2rel

Returns the relative path for the absolute path in the IO::All object.
Can take an optional argument indicating the base path.

- * canonpath

Returns the canonical path for the IO::All object.

- * case_tolerant

Returns 0 or 1 indicating whether the file system is case tolerant.
Since an active IO::All object is not needed for this function, you can
code it like:

    IO::All->case_tolerant;

or more simply:

    io->case_tolerant;

- * catdir

Concatenate the directory components together, and return a new IO::All
object representing the resulting directory.

- * catfile

Concatenate the directory and file components together, and return a new
IO::All object representing the resulting file.

    my $contents = io->catfile(qw(dir subdir file))->slurp;

This is a very portable way to read `dir/subdir/file`.

- * catpath

Concatenate the volume, directory and file components together, and
return a new IO::All object representing the resulting file.

- * curdir

Returns an IO::All object representing the current directory.

- * devnull

Returns an IO::All object representing the /dev/null file.

- * is_absolute

Returns 0 or 1 indicating whether the `name` field of the IO::All object is
an absolute path.

- * join

Same as `catfile`.

- * path

Returns a list of IO::All directory objects for each directory in your path.

- * rel2abs

Returns the absolute path for the relative path in the IO::All object. Can
take an optional argument indicating the base path.

- * rootdir

Returns an IO::All object representing the root directory on your
file system.

- * splitdir

Returns a list of the directory components of a path in an IO::All object.

- * splitpath

Returns a volume directory and file component of a path in an IO::All object.

- * tmpdir

Returns an IO::All object representing a temporary directory on your
file system.

- * updir

Returns an IO::All object representing the current parent directory.

# OPERATIONAL NOTES

- *

Each IO::All object gets reblessed into an IO::All::* object as soon as
IO::All can determine what type of object it should be. Sometimes it gets
reblessed more than once:

    my $io = io('mydbm.db');
    $io->dbm('DB_File');
    $io->{foo} = 'bar';

In the first statement, $io has a reference value of 'IO::All::File', if
`mydbm.db` exists. In the second statement, the object is reblessed into
class 'IO::All::DBM'.

- *

An IO::All object will automatically be opened as soon as there is
enough contextual information to know what type of object it is, and
what mode it should be opened for. This is usually when the first read
or write operation is invoked but might be sooner.

- *

The mode for an object to be opened with is determined heuristically
unless specified explicitly.

- *

For input, IO::All objects will automatically be closed after EOF (or
EOD). For output, the object closes when it goes out of scope.

To keep input objects from closing at EOF, do this:

    $io->autoclose(0);

- * 

You can always call `open` and `close` explicitly, if you need that
level of control. To test if an object is currently open, use the
`is_open` method.

- *

Overloaded operations return the target object, if one exists.

This would set `$xxx` to the IO::All object:

    my $xxx = $contents > io('file.txt');

While this would set `$xxx` to the content string:

    my $xxx = $contents < io('file.txt');

# STABILITY

The goal of the IO::All project is to continually refine the module
to be as simple and consistent to use as possible. Therefore, in the
early stages of the project, I will not hesitate to break backwards
compatibility with other versions of IO::All if I can find an easier
and clearer way to do a particular thing.

IO is tricky stuff. There is definitely more work to be done. On the
other hand, this module relies heavily on very stable existing IO
modules; so it may work fairly well.

I am sure you will find many unexpected "features". Please send all
problems, ideas and suggestions to ingy@cpan.org.

## Known Bugs and Deficiencies

Not all possible combinations of objects and methods have been tested.
There are many many combinations. All of the examples have been tested.
If you find a bug with a particular combination of calls, let me know.

If you call a method that does not make sense for a particular object,
the result probably won't make sense. Little attempt is made to check
for improper usage.

# SEE ALSO

IO::Handle, IO::File, IO::Dir, IO::Socket, IO::String, File::Spec,
File::Path, File::ReadBackwards, Tie::File

# CREDITS

A lot of people have sent in suggestions, that have become a part of
IO::All. Thank you.

Special thanks to Ian Langworth for continued testing and patching.

Thank you Simon Cozens for tipping me off to the overloading possibilities.

Finally, thanks to Autrijus Tang, for always having one more good idea.

(It seems IO::All of it to a lot of people!)

# REPOSITORY AND COMMUNITY

The IO::All module can be found on CPAN and on GitHub:
<http://github.com/ingydotnet/io-all-pm>.

Please join the IO::All discussion on #io-all on irc.perl.org.

# AUTHOR

Ingy döt Net <ingy@cpan.org>

# COPYRIGHT

Copyright (c) 2004. Brian Ingerson.

Copyright (c) 2006, 2008, 2010. Ingy döt Net.

This program is free software; you can redistribute it and/or modify it
under the same terms as Perl itself.

See <http://www.perl.com/perl/misc/Artistic.html>