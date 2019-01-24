# Swift FireBaseを利用したChatアプリの作成

## 本章の目的：

- FireBaseの環境構築
- 

***
FireBaseの環境構築をしていきます。

#### 5 info.plistの設定
図5

```
![図5. 設定画面](1-5.png)

```
グーグルのページからダウンロードした,info.plistの
REVERSED_CLIENT_ID key
をコピーしてURL Schemesに貼り付けます。

こんな感じになります。

```
![図5-2. 設定画面](5-2.png)

```
 
  AppDelegate.swiftに書き加えます。
  アプリ起動時に呼ばれるメソッドです。

```swift
  func application(_ application: UIApplication, didFinishLaunchingWithOptions
    launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    
    //ここから
    FirebaseApp.configure()
    GIDSignIn.sharedInstance().clientID = FirebaseApp.app()?.options.clientID
    GIDSignIn.sharedInstance().delegate = self
    //ここまで
    
    return true
  }
```

はじめから記載されているsignメソッドに　サインイン時のメソッドを書き加えます。

```swift
  func sign(_ signIn: GIDSignIn!, didSignInFor user: GIDGoogleUser!, withError error: Error?) {
    if let error = error {
      print("Error \(error)")
      return
    }

    //ここから
    guard let authentication = user.authentication else { return}
    let credential = GoogleAuthProvider.credential(withIDToken: authentication.idToken, accessToken: authentication.accessToken)
    
    Auth.auth().signIn(with: credential){ (user, error) in
        if let error = error{
            print(error)
            return
        }
    }
    //ここまで

  }
```

何をしているのか見ていきましょう。
```swift
guard let authentication = user.authentication else { return}
```
guard節です。条件が偽なら{ }が走ります。
user.authenticationが存在するなら authentication という変数に代入して下さい
そうでないなら、ここでおしまい。

```swift
    Auth.auth().signIn(with: credential){ (user, error) in
        if let error = error{
            print(error)
            return
        }
    }
```
Authクラスのauth()　インスタンスを作って .signIn プロパティを 
先程の credential　を引数にしてサインインしてください。  
<br />
そこからクロージャーになっていますね。
<br />
  (user, error) が引数で、in　以下が処理です。
  errorがあったら、errorに代入してログに出力して下さい
  そうでないなら何もしない。
  つまりエラーがなければ、ログインできるということです。


***
これで、AppDelegateの実装は終わりです。
次はログイン画面に実装していきます。

## SignInViewControllerの実装

```swift
 var handle: AuthStateDidChangeListenerHandle?

  override func viewDidLoad() {
    super.viewDidLoad()
//ここから
    GIDSignIn.sharedInstance().uiDelegate = self
    GIDSignIn.sharedInstance()?.signInSilently()
    handle = Auth.auth().addStateDidChangeListener(){(auth, user) in
        if user != nil{
            MeasurementHelper.sendLoginEvent()
            self.performSegue(withIdentifier: Constants.Segues.SignInToFp, sender: nil)
            
        }
    }
    //ここまで
  }
```
***
解説

```
GIDSignIn.sharedInstance().uiDelegate = self
```
GIDSignInでインスタンスを生成しています。
これはグーグル関連の、ログイン、ログアウト関係を処理するクラスです。
handleにはaddStateDidChangeListener関数を読んでクロージャーで処理しています。
引数は同じく(auth, user)で
userがnilでなければ、存在したら
performSegueで画面遷移しなさいということです。
Constants.Segues.SignInToFp　はただのリテラル（文字）が入っている変数です。
実際の開発では、文字入力はエラーが起きやすいので、このように変数に代入して使うことが多いです。


```
![図6. 設定画面](1-6.png)

```

その後は　deinitしています。
インスタンスの後処理です。
これはARCというメモリ上に使われていないインスタンスを処理する機能により、勝手に呼ばれます。
しかし認証関係は、通常のインスタンスと違い、大切なので、きちんと記載しておきます。
    
```swift
  deinit {
        if let handle = handle{
        Auth.auth().removeStateDidChangeListener(handle)
        }
    }
```
もし、使われていない　handle　インスタンスがあったら、removeStateDidChangeListener　メソッドによって削除しましょう
ということです。
initの反対ですね。

***
#### 次の章から、データベースを作っていきます。

