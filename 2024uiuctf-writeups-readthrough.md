# Fare evasion 

## Writeup: https://www.su-cvestone.cn/445/#总结考察点

The soruce code available after `ctrl + u`:

```html
  <script>
    async function pay() {
      // i could not get sqlite to work on the frontend :(
      /*
        db.each(`SELECT * FROM keys WHERE kid = '${md5(headerKid)}'`, (err, row) => {
        ???????
       */
      const r = await fetch("/pay", { method: "POST" });
      const j = await r.json();
      document.getElementById("alert").classList.add("opacity-100");
      // todo: convert md5 to hex string instead of latin1??
      document.getElementById("alert").innerText = j["message"];
      setTimeout(() => { document.getElementById("alert").classList.remove("opacity-100") }, 5000);
    }
  </script>
```
At first glance this seems like an **sql injection**. However, it is hashed with **MD5** without being used in a query

After intercepting the data with burp suite, you notice there are `cookie values` available. Using the value from `Cookie: access_token` we can put into https://jwt.io.

We notice that there is `kid: passenger_key` under `HEADER` and `type: passenger` under PAYLOAD. What we are left to find is the `SIGNATURE`.

After some research and stumbling upon this website: `https://book.hacktricks.xyz/pentesting-web/sql-injection#raw-hash-authentication-bypass`, there is a similar text under `Raw hash authentication Bypass` 

```
"SELECT * FROM admin WHERE pass = '".md5($password,true)."'"
```

MD5 is vulnerable to `ffifdyop`, Now we can change the kid value to `kid: ffifdyop`.

Sending the new cookie using repeater (burp suite) we get a similar message. 

```
    {
        "message":"Key isn't passenger or conductor. Please sign your own tickets. \nhashed \u00f4\u008c\u00f7u\u009e\u00deIB\u0090\u0005\u0084\u009fB\u00e7\u00d9+ secret: conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e\nhashed _\bR\u00f2\u001es\u00dcx\u00c9\u00c4\u0002\u00c5\u00b4\u0012\\\u00e4 secret: a_boring_passenger_signing_key_?",
        "success":false
    }

```

We can see that the `conductor_key` is now exposed. If we use `conductor_key_873affdf8cc36a592ec790fc62973d55f4bf43b321bf1ccc0514063370356d5cddb4363b4786fd072d36a25e0ab60a78b8df01bd396c7a05cccbbb3733ae3f8e` for our signature and plug the new cookie value we get the flag: `uiuctf{sigpwny_does_not_condone_turnstile_hopping!}`


## QUESTIONS: 

- what is `kid`: 
    kid is the "block" that stores a value. Kid = A, if this is true it would lead to the value. In this case it was kid = md5(headerkid)

- what is `jwt`?
    JWT is JSON web token, the value for cookie access_token in burp suite. 
  - How do you know this is a jwt?
        There are cookies and access tokens

# Useful websites: 
 - https://book.hacktricks.xyz
 - https://jwt.io
 - https://cvk.posthaven.com/sql-injection-with-raw-md5-hashes

# Log-action 

## Writeup: https://siunam321.github.io/ctf/UIUCTF-2024/Web/Log-Action/

Reading the zip file shows that the flag.txt is in log-action/backend/flag.txt. The flag could be found at `http://<back-end_IP>/flag.txt`.

If the package manager is npm. In the npm command-line tool, we can use the npm audit command to check for dependencies issues. 

Version `13.4.0` of Next.Js has a SSRF vulnerability with the CVE number: `CVE-2024-34351`

When we call a server action and it responds with a redirect, it calls asynchronous function `createRedirectRenderResult`.

Source code from the logout page, it uses `redirect("/login")`

```javascript
import Link from "next/link";
import { redirect } from "next/navigation";
import { signOut } from "@/auth";

export default function Page() {
  return (
    <>
      <h1 className="text-2xl font-bold">Log out</h1>
      <p>Are you sure you want to log out?</p>
      <Link href="/admin">
        Go back
      </Link>
      <form
        action={async () => {
          "use server";
          await signOut({ redirect: false });
          redirect("/login");
        }}
      >
        <button type="submit">Log out</button>
      </form>
    </>
  )
}
```

> Note: ngrok is not open-source since 2015. They are a private company. Virtual Machine recommended when using it. 

# Questions

- What is SSRF?
- What is ngrok?
