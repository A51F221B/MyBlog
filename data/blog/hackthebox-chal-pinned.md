---
title: Hack The Box - Pinned
date: '6-01-2025'
tags: ['hackthebox', 'android', 'ctf', 'ctf writeup', 'android security']
draft: false
summary: 
---

## Pinned
Before we start, there are a number of concepts and contexts that we need to understand. 

First is how SSL pinning is implemented in Java. There are two classes in java that are important in the context of ssl pinning.
- `KeyStore`
This class is used to store cryptographic keys and certificates. It can be used to store the public key of a server, which can then be used to verify the server's identity during an SSL handshake.
- `TrustManagerFactory`
This class is used to create a `TrustManager` that can be used to verify the identity of a server during an SSL handshake. A `TrustManager` is responsible for verifying the server's certificate and ensuring that it is valid and trusted.
- `SSLContext`
This class is used to create an SSL context, which is a container for the various components involved in an SSL handshake, including the `KeyManager` and 
`TrustManager`. The `SSLContext` can be configured to use a specific `KeyManager` and `TrustManager`, which can be used to implement SSL pinning.

### Description
The following java built-in libraries are used in the challenge :

```java
import java.io.BufferedReader; // used to read data from a file
import java.io.DataOutputStream; // used to write data to a file
import java.io.IOException; // used to handle exceptions
import java.io.InputStreamReader; // used to read data from a file
import java.net.HttpURLConnection; // used to make http requests
import java.net.URL; // used to make http requests
import java.security.KeyStore; // used to store cryptographic keys and certificates
import java.security.cert.Certificate; // used to verify the identity of a server
import java.security.cert.CertificateFactory; //used to verify the identity of a server
import java.util.ArrayList; //used to store data
import javax.crypto.Cipher; //used to encrypt and decrypt data
import javax.crypto.spec.SecretKeySpec; //used to encrypt and decrypt data
import javax.net.ssl.HttpsURLConnection; //used to make https requests
import javax.net.ssl.SSLContext; //used to create an ssl context
import javax.net.ssl.TrustManagerFactory; //used to create a trust manager
```

The code starts with the following `View.onClickListener` method

```java
 public class a implements View.OnClickListener {
        public a() {
        }

        @Override // android.view.View.OnClickListener
        public void onClick(View view) {
            MainActivity.this.r.setText("");
            try {
                MainActivity.this.w();
                MainActivity.this.x();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
```

The `try` block contains two method calls.
The first method is `w()` which is defined as follows:

```java
    public void w() {
        this.p = CertificateFactory.getInstance("X.509").generateCertificate(getResources().openRawResource(R.raw.certificate));
        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
        keyStore.load(null, null);
        keyStore.setCertificateEntry("Self signed certificate", this.p);
        TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        trustManagerFactory.init(keyStore);
        SSLContext sSLContext = SSLContext.getInstance("TLS");
        this.q = sSLContext;
        sSLContext.init(null, trustManagerFactory.getTrustManagers(), null);
    }
```

In the above code, we are loading a certificate from the `res/raw/certificate` file and adding it to a `KeyStore`. We then create a `TrustManagerFactory` and initialize it with the `KeyStore`. Finally, we create an `SSLContext` and initialize it with the `TrustManagerFactory` to only use this certificate.


The second method is `x()` which is defined as follows:

```java
    public void x() {
        HttpsURLConnection httpsURLConnection;
        TextView textView;
        String str;
        MainActivity mainActivity = this;
        HttpsURLConnection httpsURLConnection2 = (HttpsURLConnection) new URL("https://pinned.com:443/pinned.php").openConnection();
        httpsURLConnection2.setRequestMethod("POST");
        httpsURLConnection2.setRequestProperty("Content-Type", "application/x-www-form-urlencoded");
        httpsURLConnection2.setRequestProperty("Accept", "application/x-www-form-urlencoded");
        httpsURLConnection2.setRequestProperty("charset", "utf-8");
        httpsURLConnection2.setDoOutput(true);
        httpsURLConnection2.setSSLSocketFactory(mainActivity.q.getSocketFactory());
        if (mainActivity.s.getText().toString().equals("bnavarro") && mainActivity.t.getText().toString().equals("1234567890987654")) {
            StringBuilder g = c.a.a.a.a.g("uname=bnavarro&pass=");
            StringBuilder sb = new StringBuilder();
            sb.append(d.a());
            sb.append(c.b.a.b.a());
            sb.append(h.a());
            sb.append(c.a());
            sb.append(i.a());
            ArrayList arrayList = new ArrayList();
            arrayList.add("9GDFt6");
            arrayList.add("83h736");
            arrayList.add("kdiJ78");
            arrayList.add("vcbGT6");
            arrayList.add("LPGt63");
            arrayList.add("kFgde4");
            arrayList.add("5drDr4");
            arrayList.add("Y6ttr5");
            arrayList.add("444w45");
            arrayList.add("hjKd56");
            sb.append((String) arrayList.get(4));
            sb.append(e.a());
            sb.append(f.a());
            ArrayList arrayList2 = new ArrayList();
            arrayList2.add("TG7ygj");
            arrayList2.add("U8uu8i");
            arrayList2.add("gGtT56");
            arrayList2.add("84hYDG");
            arrayList2.add("yRCYDm");
            arrayList2.add("7ytr4E");
            arrayList2.add("j5jU87");
            arrayList2.add("yRCYDm");
            arrayList2.add("jd9Idu");
            arrayList2.add("kd546G");
            sb.append((String) arrayList2.get(7));
            sb.append(c.b.a.a.a());
            String sb2 = sb.toString();
            httpsURLConnection = httpsURLConnection2;
            SecretKeySpec secretKeySpec = new SecretKeySpec((String.valueOf(b.q.h.b().charAt(3)) + String.valueOf(b.q.h.b().charAt(0)) + String.valueOf(c.a().charAt(0)) + String.valueOf(c.b.a.a.a().charAt(8)).toUpperCase() + String.valueOf(h.a().charAt(1)) + String.valueOf(b.q.h.b().charAt(0)) + String.valueOf(i.a().charAt(5)).toUpperCase() + String.valueOf(c.b.a.a.a().charAt(7)) + String.valueOf(c.b.a.b.a().charAt(4)) + String.valueOf(e.a().charAt(4)) + String.valueOf(e.a().charAt(4)) + String.valueOf(i.a().charAt(5)).toUpperCase() + String.valueOf(d.a().charAt(3)) + String.valueOf(d.a().charAt(5)) + String.valueOf(h.a().charAt(1)) + String.valueOf(h.a().charAt(1))).getBytes(), g.a());
            Cipher cipher = Cipher.getInstance(g.a());
            cipher.init(2, secretKeySpec);
            g.append(new String(cipher.doFinal(Base64.decode(sb2, 0)), "utf-8"));
            mainActivity = this;
            mainActivity.o = g.toString();
            textView = mainActivity.r;
            str = "You are logged in.";
        } else {
            httpsURLConnection = httpsURLConnection2;
            StringBuilder g2 = c.a.a.a.a.g("uname=");
            g2.append(mainActivity.s.getText().toString());
            g2.append("&pass=");
            g2.append(mainActivity.t.getText().toString());
            mainActivity.o = g2.toString();
            textView = mainActivity.r;
            str = "Wrong credentials!";
        }
        textView.setText(str);
        DataOutputStream dataOutputStream = new DataOutputStream(httpsURLConnection.getOutputStream());
        try {
            dataOutputStream.writeBytes(mainActivity.o);
            dataOutputStream.flush();
            dataOutputStream.close();
            new Thread(mainActivity.new b(httpsURLConnection)).start();
        } catch (Throwable th) {
            try {
                throw th;
            } finally {
            }
        }
    }
```

In the above method, we are making a POST request to the `https://pinned.com:443/pinned.php` URL. We are also setting the `SSLSocketFactory` of the `HttpsURLConnection` to the `SSLContext` that we created in the `w()` method. This means that the `HttpsURLConnection` will use the certificate that we loaded in the `w()` method to verify the server's identity.


In the class `b` there is an interesting method :

```java
        public void run() {
            try {
                StringBuilder sb = new StringBuilder();
                sb.append(this.f1023b.getResponseCode());
                sb.append(" ");
                sb.append(this.f1023b.getResponseMessage());
                sb.append("\n");
                Certificate[] serverCertificates = this.f1023b.getServerCertificates();
                for (Certificate certificate : serverCertificates) {
                    if (!MainActivity.this.p.toString().equals(serverCertificates[0].toString())) {
                        throw new Exception("Connection is not honoured as the bundled certificate does not match the certificate data in the incoming request.");
                    }
                    MainActivity.super.runOnUiThread(new a(MainActivity.u(MainActivity.this, this.f1023b)));
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
```

In the above method, we are checking if the certificate that we loaded in the `w()` method matches the certificate that is returned by the server. If it does not match, we throw an exception. This means that if an attacker tries to intercept the request and replace the server's certificate with their own, the app will not accept the connection and will throw an exception.

Our goal here is to bypass the ssl pinning so we can intercept the flag.To do that we can use a tool like `frida` to hook into the app and modify the behavior of the `x()` method. We can replace the `if` statement that checks if the certificate matches with a statement that always returns true. This will allow us to intercept the request and get the flag.