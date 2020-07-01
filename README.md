# twitter_oauth

個人のプロダクトで動いていたコードを掻い摘んで公開していただけなのでそこそこきれいに書き直したコードをプラグインとして公開しました  
例外処理やサンプルコードはスカスカですが、使えることは確認しています。  
良かったら使ってください  
https://github.com/0maru/twitter_login  
  

Twitter Sign-in Library

    import 'package:flutter/material.dart';
    import 'package:firebase_auth/firebase_auth.dart';
    import 'package:twitter_oauth/twitter_oauth.dart';
    import 'package:webview_flutter/webview_flutter.dart';

    class TwitterOauthPage extends StatefulWidget {
      const TwitterOauthPage({Key key}) : super(key: key);

      @override
      _TwitterOauthPageState createState() => _TwitterOauthPageState();
    }

    class _TwitterOauthPageState extends State<TwitterOauthPage> {
      TwitterOauth _twitterOauth;

      @override
      void initState() {
        super.initState();
        _twitterOauth = TwitterOauth(
          'XXXXXXXXXX',
          'XXXXXXXXXXXXXXXXXXX',
          'https://XXXXXXXX.firebaseapp.com/__/auth/handler',
        );
      }

      @override
      Widget build(BuildContext context) {
        return Scaffold(
          appBar: AppBar(
            title: const Text('Sign In With Twitter'),
          ),
          body: Center(
            child: RaisedButton(
              child: const Text('Sign In With Twitter.'),
              onPressed: () async {
                final String authorizeUri =   
                  await _twitterOauth.getAuthorizeUri();
                Navigator.of(context).pushReplacement<Widget, Widget>(
                  MaterialPageRoute<Widget>(
                    builder: (BuildContext context) {
                      return TwitterWebView(
                        uri: authorizeUri,
                      );
                    },
                  ),
                );
              },
            ),
          ),
        );
      }
    }

    class TwitterWebView extends StatefulWidget {
      const TwitterWebView({Key key, this.uri}) : super(key: key);
      final String uri;
      @override
      _TwitterWebViewState createState() => _TwitterWebViewState();
    }

    class _TwitterWebViewState extends State<TwitterWebView> {
      TwitterOauth _twitterOauth;

      @override
      void initState() {
        super.initState();
        _twitterOauth = TwitterOauth(
          'XXXXXXXXXX',
          'XXXXXXXXXXXXXXXXXXX',
          'https://XXXXXXXX.firebaseapp.com/__/auth/handler',
        );
      }

      @override
      Widget build(BuildContext context) {
        return Scaffold(
          body: WebView(
            initialUrl: widget.uri,
            javascriptMode: JavascriptMode.unrestricted,
            navigationDelegate: (NavigationRequest request) {
              if (request.url.startsWith('callbackUri')) {
                final String query = request.url.split('?').last;
                if (query.contains('denied')) {
                  /// Cancel
                } else {
                  final Map<String, String> res = Uri.splitQueryString(query);
                  twitterSignin(res).then((String uid) {
                    /// Navigato to Main Page
                  });
                }
              }
              return NavigationDecision.navigate;
            },
          ),
        );
      }

      Future<String> twitterSignin(Map<String, String> token) async {
        final Map<String, String> oauthToken =
            await _twitterOauth.getAccessToken(token);
        final AuthCredential credential = TwitterAuthProvider.getCredential(
          authToken: oauthToken['oauth_token'],
          authTokenSecret: oauthToken['oauth_token_secret'],
        );
        final FirebaseUser user =
            await FirebaseAuth.instance.signInWithCredential(credential);
        return user.uid;
      }
    }
