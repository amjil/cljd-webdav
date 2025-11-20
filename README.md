# WebDAV Client - ClojureDart Project

A WebDAV client project developed with ClojureDart, supporting file upload, download, delete, and directory listing functionalities.

## Table of Contents

- [Caddy Installation and Configuration](#caddy-installation-and-configuration)
- [WebDAV Service Startup](#webdav-service-startup)
- [ClojureDart Usage](#clojuredart-usage)
- [Code Examples](#code-examples)

## Caddy Installation and Configuration

### Building Caddy with xcaddy

xcaddy is Caddy's build tool that allows you to customize Caddy compilation and add plugins.

#### Install xcaddy

```bash
# macOS
brew install xcaddy

# Or install using Go
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest
```

#### Build Caddy with WebDAV Plugin

```bash
xcaddy build --with github.com/mholt/caddy-webdav
```

After compilation, the `caddy` executable will be generated in the current directory.

#### Verify Installation

```bash
./caddy version
```

## WebDAV Service Startup

### Configuration File

The project root directory contains a `caddyfile` configuration file:

```1:15:caddyfile
:8080 {
	route /dav/* {

		basicauth {
			user $2a$14$ClNTzReiTOCTrlb1KR08KugifBSgpls3qqFkLd56cpCL0rdDOdxY.
		}

		webdav {
			root ~/webdav/
		}

	}
	
}
```

### Generate Basic Auth Password Hash

To generate a password hash for the `basicauth` directive in the caddyfile, use Caddy's built-in password hashing command:

```bash
# Generate password hash (interactive mode)
caddy hash-password

# Or generate hash directly with password
caddy hash-password --plaintext "your-password-here"
```

**Example:**
```bash
$ caddy hash-password --plaintext "123123123"
$2a$14$ClNTzReiTOCTrlb1KR08KugifBSgpls3qqFkLd56cpCL0rdDOdxY.
```

The output is a bcrypt hash that you can use in the `caddyfile`:

```caddyfile
basicauth {
    user $2a$14$ClNTzReiTOCTrlb1KR08KugifBSgpls3qqFkLd56cpCL0rdDOdxY.
}
```

**Note:** Replace `user` with your desired username and use the generated hash for the password.

### Prepare Directory

**Important:** The directory specified in the `caddyfile` must exist before starting the service. The `root` directory in the webdav configuration must be created, and you need to create a `dav` subdirectory inside it:

```bash
# Create the WebDAV root directory
mkdir -p ~/wabdav/dav
```

The directory structure should be:
- `~/webdav/` - Parent directory
- `~/webdav/dav/` - WebDAV root directory (as specified in caddyfile)

Both directories must exist and have appropriate write permissions for WebDAV operations to work correctly.

### Start Service

```bash
# Start service using the compiled caddy with absolute path
./caddy run --config /path/to/caddyfile

# Or use relative path
./caddy run --config caddyfile

# Or use system-installed caddy
caddy run --config caddyfile
```

The service will run on `http://localhost:8080/dav`.

**Default Credentials:**
- Username: `user`
- Password: `123123123`

**WebDAV Root Directory:** `~/dav/`

### Testing with curl

You can test the WebDAV service using curl commands:

#### 1. List Directory Contents (PROPFIND)

```bash
curl -v -u user:123123123 -X PROPFIND http://localhost:8080/dav/
```

#### 2. Upload File (PUT)

```bash
# Create a test file
echo "hello world" > /tmp/test.txt

# Upload the file
curl -u user:123123123 -T /tmp/test.txt http://localhost:8080/dav/test.txt
```

#### 3. Verify Upload (PROPFIND)

```bash
# List directory again to verify the file was uploaded
curl -u user:123123123 -X PROPFIND http://localhost:8080/dav/
```

#### 4. Download File (GET)

```bash
# Download the file
curl -u user:123123123 http://localhost:8080/dav/test.txt
```

#### 5. Create Directory (MKCOL)

```bash
# Create a new directory
curl -u user:123123123 -X MKCOL http://localhost:8080/dav/newdir/
```

#### 6. Delete File (DELETE)

```bash
# Delete the test file
curl -u user:123123123 -X DELETE http://localhost:8080/dav/test.txt
```

#### 7. Delete Directory (DELETE)

```bash
# Delete the directory
curl -u user:123123123 -X DELETE http://localhost:8080/dav/newdir/
```

## ClojureDart Usage

### Requirements

- Clojure CLI (clj)
- Flutter SDK
- Dart SDK

### Install Dependencies

```bash
# Install ClojureDart dependencies
clj -M:cljd prep

# Install Flutter/Dart dependencies
flutter pub get
```

### Run Project

```bash
# Run in development mode
clj -M:cljd flutter

## Code Examples

### Function Descriptions

#### 1. Create Client

```clojure
(let [client (client/create-client "http://localhost:8080/dav" "user" "123123123")]
  ;; Use client for operations
  )
```

#### 2. List Directory Contents

```clojure
(-> (client/list-dir client "/")
    (.then (fn [x] (dart:core/print x))))
```

#### 3. Upload File

```clojure
(-> (client/upload-file client "Hello, World!" "/test1.txt")
    (.then (fn [x] (dart:core/print x)))
    (.catchError (fn [error]
                   (dart:core/print (http/format-error-message (str error) "/test1.txt")))))
```

#### 4. Download File

```clojure
(-> (client/download-file client "/test.txt")
    (.then (fn [x] (dart:core/print x))))
```

#### 5. Delete File

```clojure
(-> (client/delete-file client "/test1.txt")
    (.then (fn [_] 
             (dart:core/print "File deleted successfully")))
    (.catchError (fn [error]
                   (dart:core/print (http/format-error-message (str error) "/test1.txt")))))
```

### Available Client Methods

- `client/list-dir` - List directory contents (PROPFIND)
- `client/upload-file` - Upload file (PUT)
- `client/download-file` - Download file (GET)
- `client/delete-file` - Delete file (DELETE)
- `client/make-dir` - Create directory (MKCOL)

### Error Handling

All asynchronous operations support `.catchError` for error handling, and you can use `http/format-error-message` to format error messages.

## Project Structure

```
webdav/
├── src/
│   └── webdav/
│       ├── main.cljd      # Main entry file
│       ├── client.cljd    # WebDAV client implementation
│       ├── http.cljd      # HTTP request handling
│       └── utils.cljd     # Utility functions
├── caddyfile              # Caddy configuration file
├── deps.edn               # Clojure dependency configuration
└── pubspec.yaml           # Flutter/Dart dependency configuration
```

## Notes

1. Make sure the Caddy WebDAV service is started before running the client
2. Modify the URL, username, and password in `main.cljd` to match your Caddy configuration
3. All file operations are asynchronous and require using `.then` and `.catchError` to handle results
4. The WebDAV root directory defaults to `~/webdav/`, ensure this directory exists and has write permissions

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
