
  <body>
  
 
<p>Когда незамысловатая идея игры охватывает мир.</p>

<p>Как известно, математическую игру «2048», создал итальянский разработчик Gabriele Cirulli.

Игровое поле состоит из сетки 4х4.
Игра начинается. На сцене две плитки с номиналом 2.
Передвигая, нужно сложить плитки одного «номинала».
Движение возможно в 4 стороны.
</p>

<img src="https://habrastorage.org/getpro/habr/post_images/c9e/ffc/63b/c9effc63b2b12c858c66a937188406c4.png" alt="image">

<p>Цель игры — собрать плитку с «номиналом» 2048.</p>

<img src="https://habrastorage.org/getpro/habr/post_images/1c2/138/a4f/1c2138a4fe7d931d4ab93f01f3c6d432.png" alt="image">

<h3>Реализация основной механики игры:</h3>

<div class="highlight highlight-source-java"><pre><span class="pl-k">package</span> <span class="pl-smi">com.John.a2048</span>;

<span class="pl-k">import</span> <span class="pl-smi">android.annotation.SuppressLint</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.app.Activity</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.content.SharedPreferences.Editor</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.content.pm.ActivityInfo</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.content.res.Configuration</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.os.Bundle</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.preference.PreferenceManager</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.provider.Settings</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.provider.Settings.SettingNotFoundException</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.util.Log</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.view.MotionEvent</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.view.Window</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.view.WindowManager.LayoutParams</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.webkit.WebSettings</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.webkit.WebSettings.RenderPriority</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.webkit.WebView</span>;
<span class="pl-k">import</span> <span class="pl-smi">android.widget.Toast</span>;

<span class="pl-k">import</span> <span class="pl-smi">java.util.Locale</span>;

<span class="pl-k">import</span> <span class="pl-smi">de.cketti.changelog.dialog.DialogChangeLog</span>;

<span class="pl-k">public</span> <span class="pl-k">class</span> <span class="pl-en">MainActivity</span> <span class="pl-k">extends</span> <span class="pl-e">Activity</span> {

    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-k">final</span> <span class="pl-smi">String</span> <span class="pl-c1">MAIN_ACTIVITY_TAG</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>2048_MainActivity<span class="pl-pds">"</span></span>;

    <span class="pl-k">private</span> <span class="pl-smi">WebView</span> mWebView;
    <span class="pl-k">private</span> <span class="pl-k">long</span> mLastBackPress;
    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-k">final</span> <span class="pl-k">long</span> mBackPressThreshold <span class="pl-k">=</span> <span class="pl-c1">3500</span>;
    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-k">final</span> <span class="pl-smi">String</span> <span class="pl-c1">IS_FULLSCREEN_PREF</span> <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">"</span>is_fullscreen_pref<span class="pl-pds">"</span></span>;
    <span class="pl-k">private</span> <span class="pl-k">long</span> mLastTouch;
    <span class="pl-k">private</span> <span class="pl-k">static</span> <span class="pl-k">final</span> <span class="pl-k">long</span> mTouchThreshold <span class="pl-k">=</span> <span class="pl-c1">2000</span>;
    <span class="pl-k">private</span> <span class="pl-smi">Toast</span> pressBackToast;

    <span class="pl-k">@SuppressLint</span>({<span class="pl-s"><span class="pl-pds">"</span>SetJavaScriptEnabled<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>ShowToast<span class="pl-pds">"</span></span>, <span class="pl-s"><span class="pl-pds">"</span>ClickableViewAccessibility<span class="pl-pds">"</span></span>})
    <span class="pl-k">@Override</span>
    <span class="pl-k">protected</span> <span class="pl-k">void</span> <span class="pl-en">onCreate</span>(<span class="pl-smi">Bundle</span> <span class="pl-v">savedInstanceState</span>) {
        <span class="pl-c1">super</span><span class="pl-k">.</span>onCreate(savedInstanceState);

        <span class="pl-c"><span class="pl-c">//</span> Don't show an action bar or title</span>
        requestWindowFeature(<span class="pl-smi">Window</span><span class="pl-c1"><span class="pl-k">.</span>FEATURE_NO_TITLE</span>);

        <span class="pl-c"><span class="pl-c">//</span> Enable hardware acceleration</span>
        getWindow()<span class="pl-k">.</span>setFlags(<span class="pl-smi">LayoutParams</span><span class="pl-c1"><span class="pl-k">.</span>FLAG_HARDWARE_ACCELERATED</span>,
                <span class="pl-smi">LayoutParams</span><span class="pl-c1"><span class="pl-k">.</span>FLAG_HARDWARE_ACCELERATED</span>);

        <span class="pl-c"><span class="pl-c">//</span> Apply previous setting about showing status bar or not</span>
        applyFullScreen(isFullScreen());

        <span class="pl-c"><span class="pl-c">//</span> Check if screen rotation is locked in settings</span>
        <span class="pl-k">boolean</span> isOrientationEnabled <span class="pl-k">=</span> <span class="pl-c1">false</span>;
        <span class="pl-k">try</span> {
            isOrientationEnabled <span class="pl-k">=</span> <span class="pl-smi">Settings</span><span class="pl-k">.</span><span class="pl-smi">System</span><span class="pl-k">.</span>getInt(getContentResolver(),
                    <span class="pl-smi">Settings</span><span class="pl-k">.</span><span class="pl-smi">System</span><span class="pl-c1"><span class="pl-k">.</span>ACCELEROMETER_ROTATION</span>) <span class="pl-k">==</span> <span class="pl-c1">1</span>;
        } <span class="pl-k">catch</span> (<span class="pl-smi">SettingNotFoundException</span> e) {
            <span class="pl-smi">Log</span><span class="pl-k">.</span>d(<span class="pl-c1">MAIN_ACTIVITY_TAG</span>, <span class="pl-s"><span class="pl-pds">"</span>Settings could not be loaded<span class="pl-pds">"</span></span>);
        }

        <span class="pl-c"><span class="pl-c">//</span> If rotation isn't locked and it's a LARGE screen then add orientation changes based on sensor</span>
        <span class="pl-k">int</span> screenLayout <span class="pl-k">=</span> getResources()<span class="pl-k">.</span>getConfiguration()<span class="pl-k">.</span>screenLayout
                <span class="pl-k">&amp;</span> <span class="pl-smi">Configuration</span><span class="pl-c1"><span class="pl-k">.</span>SCREENLAYOUT_SIZE_MASK</span>;
        <span class="pl-k">if</span> (((screenLayout <span class="pl-k">==</span> <span class="pl-smi">Configuration</span><span class="pl-c1"><span class="pl-k">.</span>SCREENLAYOUT_SIZE_LARGE</span>)
                <span class="pl-k">||</span> (screenLayout <span class="pl-k">==</span> <span class="pl-smi">Configuration</span><span class="pl-c1"><span class="pl-k">.</span>SCREENLAYOUT_SIZE_XLARGE</span>))
                <span class="pl-k">&amp;&amp;</span> isOrientationEnabled) {
            setRequestedOrientation(<span class="pl-smi">ActivityInfo</span><span class="pl-c1"><span class="pl-k">.</span>SCREEN_ORIENTATION_SENSOR</span>);
        }

        setContentView(<span class="pl-smi">R</span><span class="pl-k">.</span>layout<span class="pl-k">.</span>activity_main);

        <span class="pl-smi">DialogChangeLog</span> changeLog <span class="pl-k">=</span> <span class="pl-smi">DialogChangeLog</span><span class="pl-k">.</span>newInstance(<span class="pl-c1">this</span>);
        <span class="pl-k">if</span> (changeLog<span class="pl-k">.</span>isFirstRun()) {
            changeLog<span class="pl-k">.</span>getLogDialog()<span class="pl-k">.</span>show();
        }

        <span class="pl-c"><span class="pl-c">//</span> Load webview with game</span>
        mWebView <span class="pl-k">=</span> findViewById(<span class="pl-smi">R</span><span class="pl-k">.</span>id<span class="pl-k">.</span>mainWebView);
        <span class="pl-smi">WebSettings</span> settings <span class="pl-k">=</span> mWebView<span class="pl-k">.</span>getSettings();
        settings<span class="pl-k">.</span>setJavaScriptEnabled(<span class="pl-c1">true</span>);
        settings<span class="pl-k">.</span>setDomStorageEnabled(<span class="pl-c1">true</span>);
        settings<span class="pl-k">.</span>setDatabaseEnabled(<span class="pl-c1">true</span>);
        settings<span class="pl-k">.</span>setRenderPriority(<span class="pl-smi">RenderPriority</span><span class="pl-c1"><span class="pl-k">.</span>HIGH</span>);
        settings<span class="pl-k">.</span>setDatabasePath(getFilesDir()<span class="pl-k">.</span>getParentFile()<span class="pl-k">.</span>getPath() <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">"</span>/databases<span class="pl-pds">"</span></span>);

        <span class="pl-c"><span class="pl-c">//</span> If there is a previous instance restore it in the webview</span>
        <span class="pl-k">if</span> (savedInstanceState <span class="pl-k">!=</span> <span class="pl-c1">null</span>) {
            mWebView<span class="pl-k">.</span>restoreState(savedInstanceState);
        } <span class="pl-k">else</span> {
            <span class="pl-c"><span class="pl-c">//</span> Load webview with current Locale language</span>
            mWebView<span class="pl-k">.</span>loadUrl(<span class="pl-s"><span class="pl-pds">"</span>file:///android_asset/2048/index.html?lang=<span class="pl-pds">"</span></span> <span class="pl-k">+</span> <span class="pl-smi">Locale</span><span class="pl-k">.</span>getDefault()<span class="pl-k">.</span>getLanguage());
        }

        <span class="pl-smi">Toast</span><span class="pl-k">.</span>makeText(getApplication(), <span class="pl-smi">R</span><span class="pl-k">.</span>string<span class="pl-k">.</span>toggle_fullscreen, <span class="pl-smi">Toast</span><span class="pl-c1"><span class="pl-k">.</span>LENGTH_SHORT</span>)<span class="pl-k">.</span>show();
        <span class="pl-c"><span class="pl-c">//</span> Set fullscreen toggle on webview LongClick</span>
        mWebView<span class="pl-k">.</span>setOnTouchListener((v, event) <span class="pl-k">-</span><span class="pl-k">&gt;</span> {
            <span class="pl-c"><span class="pl-c">//</span> Implement a long touch action by comparing</span>
            <span class="pl-c"><span class="pl-c">//</span> time between action up and action down</span>
            <span class="pl-k">long</span> currentTime <span class="pl-k">=</span> <span class="pl-smi">System</span><span class="pl-k">.</span>currentTimeMillis();
            <span class="pl-k">if</span> ((event<span class="pl-k">.</span>getAction() <span class="pl-k">==</span> <span class="pl-smi">MotionEvent</span><span class="pl-c1"><span class="pl-k">.</span>ACTION_UP</span>)
                    <span class="pl-k">&amp;&amp;</span> (<span class="pl-smi">Math</span><span class="pl-k">.</span>abs(currentTime <span class="pl-k">-</span> mLastTouch) <span class="pl-k">&gt;</span> mTouchThreshold)) {
                <span class="pl-k">boolean</span> toggledFullScreen <span class="pl-k">=</span> <span class="pl-k">!</span>isFullScreen();
                saveFullScreen(toggledFullScreen);
                applyFullScreen(toggledFullScreen);
            } <span class="pl-k">else</span> <span class="pl-k">if</span> (event<span class="pl-k">.</span>getAction() <span class="pl-k">==</span> <span class="pl-smi">MotionEvent</span><span class="pl-c1"><span class="pl-k">.</span>ACTION_DOWN</span>) {
                mLastTouch <span class="pl-k">=</span> currentTime;
            }
            <span class="pl-c"><span class="pl-c">//</span> return so that the event isn't consumed but used</span>
            <span class="pl-c"><span class="pl-c">//</span> by the webview as well</span>
            <span class="pl-k">return</span> <span class="pl-c1">false</span>;
        });

        pressBackToast <span class="pl-k">=</span> <span class="pl-smi">Toast</span><span class="pl-k">.</span>makeText(getApplicationContext(), <span class="pl-smi">R</span><span class="pl-k">.</span>string<span class="pl-k">.</span>press_back_again_to_exit,
                <span class="pl-smi">Toast</span><span class="pl-c1"><span class="pl-k">.</span>LENGTH_SHORT</span>);
    }

    <span class="pl-k">@Override</span>
    <span class="pl-k">protected</span> <span class="pl-k">void</span> <span class="pl-en">onResume</span>() {
        <span class="pl-c1">super</span><span class="pl-k">.</span>onResume();
        mWebView<span class="pl-k">.</span>loadUrl(<span class="pl-s"><span class="pl-pds">"</span>file:///android_asset/2048/index.html?lang=<span class="pl-pds">"</span></span> <span class="pl-k">+</span> <span class="pl-smi">Locale</span><span class="pl-k">.</span>getDefault()<span class="pl-k">.</span>getLanguage());
    }

    <span class="pl-k">@Override</span>
    <span class="pl-k">protected</span> <span class="pl-k">void</span> <span class="pl-en">onSaveInstanceState</span>(<span class="pl-smi">Bundle</span> <span class="pl-v">outState</span>) {
        mWebView<span class="pl-k">.</span>saveState(outState);
    }

    <span class="pl-c"><span class="pl-c">/**</span></span>
<span class="pl-c">     * Saves the full screen setting in the SharedPreferences</span>
<span class="pl-c">     *</span>
<span class="pl-c">     * <span class="pl-k">@param</span> isFullScreen boolean value</span>
<span class="pl-c">     <span class="pl-c">*/</span></span>

    <span class="pl-k">private</span> <span class="pl-k">void</span> <span class="pl-en">saveFullScreen</span>(<span class="pl-k">boolean</span> <span class="pl-v">isFullScreen</span>) {
        <span class="pl-c"><span class="pl-c">//</span> save in preferences</span>
        <span class="pl-smi">Editor</span> editor <span class="pl-k">=</span> <span class="pl-smi">PreferenceManager</span><span class="pl-k">.</span>getDefaultSharedPreferences(<span class="pl-c1">this</span>)<span class="pl-k">.</span>edit();
        editor<span class="pl-k">.</span>putBoolean(<span class="pl-c1">IS_FULLSCREEN_PREF</span>, isFullScreen);
        editor<span class="pl-k">.</span>apply();
    }

    <span class="pl-k">private</span> <span class="pl-k">boolean</span> <span class="pl-en">isFullScreen</span>() {
        <span class="pl-k">return</span> <span class="pl-smi">PreferenceManager</span><span class="pl-k">.</span>getDefaultSharedPreferences(<span class="pl-c1">this</span>)<span class="pl-k">.</span>getBoolean(<span class="pl-c1">IS_FULLSCREEN_PREF</span>,
                <span class="pl-c1">true</span>);
    }

    <span class="pl-c"><span class="pl-c">/**</span></span>
<span class="pl-c">     * Toggles the activity's fullscreen mode by setting the corresponding window flag</span>
<span class="pl-c">     *</span>
<span class="pl-c">     * <span class="pl-k">@param</span> isFullScreen boolean value</span>
<span class="pl-c">     <span class="pl-c">*/</span></span>
    <span class="pl-k">private</span> <span class="pl-k">void</span> <span class="pl-en">applyFullScreen</span>(<span class="pl-k">boolean</span> <span class="pl-v">isFullScreen</span>) {
        <span class="pl-k">if</span> (isFullScreen) {
            getWindow()<span class="pl-k">.</span>clearFlags(<span class="pl-smi">LayoutParams</span><span class="pl-c1"><span class="pl-k">.</span>FLAG_FULLSCREEN</span>);
        } <span class="pl-k">else</span> {
            getWindow()<span class="pl-k">.</span>setFlags(<span class="pl-smi">LayoutParams</span><span class="pl-c1"><span class="pl-k">.</span>FLAG_FULLSCREEN</span>,
                    <span class="pl-smi">LayoutParams</span><span class="pl-c1"><span class="pl-k">.</span>FLAG_FULLSCREEN</span>);
        }
    }

    <span class="pl-c"><span class="pl-c">/**</span></span>
<span class="pl-c">     * Prevents app from closing on pressing back button accidentally.</span>
<span class="pl-c">     * mBackPressThreshold specifies the maximum delay (ms) between two consecutive backpress to</span>
<span class="pl-c">     * quit the app.</span>
<span class="pl-c">     <span class="pl-c">*/</span></span>

    <span class="pl-k">@Override</span>
    <span class="pl-k">public</span> <span class="pl-k">void</span> <span class="pl-en">onBackPressed</span>() {
        <span class="pl-k">long</span> currentTime <span class="pl-k">=</span> <span class="pl-smi">System</span><span class="pl-k">.</span>currentTimeMillis();
        <span class="pl-k">if</span> (<span class="pl-smi">Math</span><span class="pl-k">.</span>abs(currentTime <span class="pl-k">-</span> mLastBackPress) <span class="pl-k">&gt;</span> mBackPressThreshold) {
            pressBackToast<span class="pl-k">.</span>show();
            mLastBackPress <span class="pl-k">=</span> currentTime;
        } <span class="pl-k">else</span> {
            pressBackToast<span class="pl-k">.</span>cancel();
            <span class="pl-c1">super</span><span class="pl-k">.</span>onBackPressed();
        }
    }
}
</pre></div>


<p>Исходный код: https://github.com/drJohnForever/John.github.io</p>



  </body>
</html>
