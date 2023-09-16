# Flutter_doc_CokBK_Net_Delete_data_on_the_internet
 https://docs.flutter.dev/cookbook/networking/delete-data
Delete data on the internet
===========================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Networking](https://docs.flutter.dev/cookbook/networking)
3.  [Delete data on the internet](https://docs.flutter.dev/cookbook/networking/delete-data)

This recipe covers how to delete data over the internet using the `http` package.

This recipe uses the following steps:

1.  Add the `http` package.
2.  Delete data on the server.
3.  Update the screen.

[](https://docs.flutter.dev/cookbook/networking/delete-data#1-add-the-http-package)1\. Add the `http` package
-------------------------------------------------------------------------------------------------------------

To add the `http` package as a dependency, run `flutter pub add`:

content_copy

```
$ flutter pub add http

```

Import the `http` package.

content_copy

```
import 'package:http/http.dart' as http;
```

[](https://docs.flutter.dev/cookbook/networking/delete-data#2-delete-data-on-the-server)2\. Delete data on the server
---------------------------------------------------------------------------------------------------------------------

This recipe covers how to delete an album from the [JSONPlaceholder](https://jsonplaceholder.typicode.com/) using the `http.delete()` method. Note that this requires the `id` of the album that you want to delete. For this example, use something you already know, for example `id = 1`.

content_copy

```
Future<http.Response> deleteAlbum(String id) async {
  final http.Response response = await http.delete(
    Uri.parse('https://jsonplaceholder.typicode.com/albums/$id'),
    headers: <String, String>{
      'Content-Type': 'application/json; charset=UTF-8',
    },
  );

  return response;
}
```

The `http.delete()` method returns a `Future` that contains a `Response`.

-   [`Future`](https://api.flutter.dev/flutter/dart-async/Future-class.html) is a core Dart class for working with async operations. A Future object represents a potential value or error that will be available at some time in the future.
-   The `http.Response` class contains the data received from a successful http call.
-   The `deleteAlbum()` method takes an `id` argument that is needed to identify the data to be deleted from the server.

[](https://docs.flutter.dev/cookbook/networking/delete-data#3-update-the-screen)3\. Update the screen
-----------------------------------------------------------------------------------------------------

In order to check whether the data has been deleted or not, first fetch the data from the [JSONPlaceholder](https://jsonplaceholder.typicode.com/) using the `http.get()` method, and display it in the screen. (See the [Fetch Data](https://docs.flutter.dev/cookbook/networking/fetch-data) recipe for a complete example.) You should now have a Delete Data button that, when pressed, calls the `deleteAlbum()` method.

content_copy

```
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  children: <Widget>[
    Text(snapshot.data?.title ?? 'Deleted'),
    ElevatedButton(
      child: const Text('Delete Data'),
      onPressed: () {
        setState(() {
          _futureAlbum =
              deleteAlbum(snapshot.data!.id.toString());
        });
      },
    ),
  ],
);
```

Now, when you click on the *Delete Data* button, the `deleteAlbum()` method is called and the id you are passing is the id of the data that you retrieved from the internet. This means you are going to delete the same data that you fetched from the internet.

### [](https://docs.flutter.dev/cookbook/networking/delete-data#returning-a-response-from-the-deletealbum-method)Returning a response from the deleteAlbum() method

Once the delete request has been made, you can return a response from the `deleteAlbum()` method to notify our screen that the data has been deleted.

content_copy

```
Future<Album> deleteAlbum(String id) async {
  final http.Response response = await http.delete(
    Uri.parse('https://jsonplaceholder.typicode.com/albums/$id'),
    headers: <String, String>{
      'Content-Type': 'application/json; charset=UTF-8',
    },
  );

  if (response.statusCode == 200) {
    // If the server did return a 200 OK response,
    // then parse the JSON. After deleting,
    // you'll get an empty JSON `{}` response.
    // Don't return `null`, otherwise `snapshot.hasData`
    // will always return false on `FutureBuilder`.
    return Album.fromJson(jsonDecode(response.body));
  } else {
    // If the server did not return a "200 OK response",
    // then throw an exception.
    throw Exception('Failed to delete album.');
  }
}
```

`FutureBuilder()` now rebuilds when it receives a response. Since the response won't have any data in its body if the request was successful, the `Album.fromJson()` method creates an instance of the `Album` object with a default value (`null` in our case). This behavior can be altered in any way you wish.

That's all! Now you've got a function that deletes the data from the internet.

[](https://docs.flutter.dev/cookbook/networking/delete-data#complete-example)Complete example
---------------------------------------------------------------------------------------------

content_copy

```
import 'dart:async';
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

Future<Album> fetchAlbum() async {
  final response = await http.get(
    Uri.parse('https://jsonplaceholder.typicode.com/albums/1'),
  );

  if (response.statusCode == 200) {
    // If the server did return a 200 OK response, then parse the JSON.
    return Album.fromJson(jsonDecode(response.body));
  } else {
    // If the server did not return a 200 OK response, then throw an exception.
    throw Exception('Failed to load album');
  }
}

Future<Album> deleteAlbum(String id) async {
  final http.Response response = await http.delete(
    Uri.parse('https://jsonplaceholder.typicode.com/albums/$id'),
    headers: <String, String>{
      'Content-Type': 'application/json; charset=UTF-8',
    },
  );

  if (response.statusCode == 200) {
    // If the server did return a 200 OK response,
    // then parse the JSON. After deleting,
    // you'll get an empty JSON `{}` response.
    // Don't return `null`, otherwise `snapshot.hasData`
    // will always return false on `FutureBuilder`.
    return Album.fromJson(jsonDecode(response.body));
  } else {
    // If the server did not return a "200 OK response",
    // then throw an exception.
    throw Exception('Failed to delete album.');
  }
}

class Album {
  final int? id;
  final String? title;

  const Album({this.id, this.title});

  factory Album.fromJson(Map<String, dynamic> json) {
    return Album(
      id: json['id'],
      title: json['title'],
    );
  }
}

void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() {
    return _MyAppState();
  }
}

class _MyAppState extends State<MyApp> {
  late Future<Album> _futureAlbum;

  @override
  void initState() {
    super.initState();
    _futureAlbum = fetchAlbum();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Delete Data Example',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Delete Data Example'),
        ),
        body: Center(
          child: FutureBuilder<Album>(
            future: _futureAlbum,
            builder: (context, snapshot) {
              // If the connection is done,
              // check for response data or an error.
              if (snapshot.connectionState == ConnectionState.done) {
                if (snapshot.hasData) {
                  return Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                      Text(snapshot.data?.title ?? 'Deleted'),
                      ElevatedButton(
                        child: const Text('Delete Data'),
                        onPressed: () {
                          setState(() {
                            _futureAlbum =
                                deleteAlbum(snapshot.data!.id.toString());
                          });
                        },
                      ),
                    ],
                  );
                } else if (snapshot.hasError) {
                  return Text('${snapshot.error}');
                }
              }

              // By default, show a loading spinner.
              return const CircularProgressIndicator();
            },
          ),
        ),
      ),
    );
  }
}
```
