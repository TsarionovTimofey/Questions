# Может ли быть два MainActivity❓

## Ответ:

Да, если двум активити указать интент фильтры action MAIN, category LAUNCHER

```
...
<activity
    android:name=".MainActivity2"
    android:exported="true"
    android:theme="@style/Theme.Notes">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>

<activity
    android:name=".MainActivity"
    android:exported="true"
    android:theme="@style/Theme.Notes">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
...
```

### Дополнительная информация:

https://developer.android.com/guide/topics/manifest/manifest-intro