1. **アプリ** プロジェクトで、`AndroidManifest.xml` ファイルを開きます。続く 2 つの手順では、コード内の *`**my_app_package**`* を、プロジェクトのアプリケーション パッケージの名前 (`manifest` タグの `package` 属性の値) に置き換えます。
2. 次の新しいアクセス許可を、既存の `uses-permission` 要素の後に追加します。
   
        <permission android:name="**my_app_package**.permission.C2D_MESSAGE"
            android:protectionLevel="signature" />
        <uses-permission android:name="**my_app_package**.permission.C2D_MESSAGE" />
        <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
        <uses-permission android:name="android.permission.GET_ACCOUNTS" />
        <uses-permission android:name="android.permission.WAKE_LOCK" />
3. `application` 開始タグの後に次のコードを追加します。
   
        <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
                                         android:permission="com.google.android.c2dm.permission.SEND">
            <intent-filter>
                <action android:name="com.google.android.c2dm.intent.RECEIVE" />
                <category android:name="**my_app_package**" />
            </intent-filter>
        </receiver>
4. ファイル *ToDoActivity.java* を開き、次の import ステートメントを追加します。
   
        import com.microsoft.windowsazure.notifications.NotificationsManager;
5. 次のプライベート変数をクラスに追加します。*`<PROJECT_NUMBER>`* を、最初の手順で Google によってアプリケーションに割り当てられたプロジェクト番号に置き換えます。
   
        public static final String SENDER_ID = "<PROJECT_NUMBER>";
6. *MobileServiceClient* の定義を **private** から **public static** に変更し、次のようにします。
   
        public static MobileServiceClient mClient;
7. 次に、通知を処理する新しいクラスを追加する必要があります。プロジェクト エクスプローラーで、**src**、**main**、**java** ノードを開き、パッケージ名のノードを右クリックして、**[新規]**、**[Java クラス]** の順にクリックします。
8. **[名前]** に「`MyHandler`」と入力して、**[OK]** をクリックします。

    ![](./media/mobile-services-android-get-started-push/android-studio-create-class.png)


1. MyHandler ファイルで、クラス宣言を以下の内容と置き換えます。
   
        public class MyHandler extends NotificationsHandler {
2. 次の `MyHandler` クラスの import ステートメントを追加します。
   
       import com.microsoft.windowsazure.notifications.NotificationsHandler;
       import android.app.NotificationManager;
       import android.app.PendingIntent;
       import android.content.Context;
       import android.content.Intent;
       import android.os.AsyncTask;
       import android.os.Bundle;
       import android.support.v4.app.NotificationCompat;
3. 次に、`MyHandler` クラスにこのメンバーを追加します。
   
       public static final int NOTIFICATION_ID = 1;
4. `MyHandler` クラスで、次のコードを追加して、**onRegistered** メソッドをオーバーライドします。このコードにより、デバイスがMobile Services の通知ハブに登録されます。
   
       @Override
       public void onRegistered(Context context,  final String gcmRegistrationId) {
           super.onRegistered(context, gcmRegistrationId);
   
           new AsyncTask<Void, Void, Void>() {
   
               protected Void doInBackground(Void... params) {
                   try {
                       ToDoActivity.mClient.getPush().register(gcmRegistrationId);
                       return null;
                   }
                   catch(Exception e) {
                       // handle error                
                   }
                   return null;              
               }
           }.execute();
       }
5. `MyHandler` クラスで、次のコードを追加して、**onReceive** メソッドをオーバーライドします。このコードは、通知を受信したときにその通知を表示します。
   
       @Override
       public void onReceive(Context context, Bundle bundle) {
               String msg = bundle.getString("message");
   
               PendingIntent contentIntent = PendingIntent.getActivity(context,
                       0, // requestCode
                       new Intent(context, ToDoActivity.class),
                       0); // flags
   
               Notification notification = new NotificationCompat.Builder(context)
                       .setSmallIcon(R.drawable.ic_launcher)
                       .setContentTitle("Notification Hub Demo")
                       .setStyle(new NotificationCompat.BigTextStyle().bigText(msg))
                       .setContentText(msg)
                       .setContentIntent(contentIntent)
                       .build();
   
               NotificationManager notificationManager = (NotificationManager)
                       context.getSystemService(Context.NOTIFICATION_SERVICE);
               notificationManager.notify(NOTIFICATION_ID, notification);
       }
6. TodoActivity.java ファイルに戻り、*ToDoActivity* クラスの **onCreate** メソッドを更新して、通知のハンドラー クラスを登録します。*MobileServiceClient* がインスタンス化された後に、このコードを追加してください。

        NotificationsManager.handleNotifications(this, SENDER_ID, MyHandler.class);

    これで、アプリケーションがプッシュ通知をサポートするように更新されました。

<!---HONumber=AcomDC_0309_2016-->