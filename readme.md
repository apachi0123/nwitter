# 602277120 정희헌

## 2022 06 28 15주차

.
1.npm install uuid

2.Home.js import 추가

```javascript
import { v4 as uuidv4 } from "uuid";
```

3.Home.js 사진 관련 코드 (console.log 제외)

```javascript
const attachmentRef = storageService.ref().child(`${userObj.uid}/$(uuidv4())`);
const response = await attachmentRef.putString(attachment, "data_url");
console.log(await response.ref.getDownloadURL);
```

4.Home.js 사진 관련 코드 (완성)

```javascript
const onSubmit = async (e) => {
  e.preventDefault();
  const attachmentRef = storageService
    .ref()
    .child(`${userObj.uid}/$(uuidv4())`);
  const response = await attachmentRef.putString(attachment, "data_url");
  const attachmentUrl = await response.ref.getDownloadURL();
  await dbService.collection("nweets").add({
    text: nweet,
    createdAt: Date.now(),
    createdID: userObj.uid,
    attachmentUrl,
  });
  setNweet("");
  setAttachment("");
};
```

5.Nweet.js 이미지 관련 코드 추가

```javascript
{
  nweetObj.attachmentUrl && (
    <img src={nweetObj.attachmentUrl} width="50px" height="50px" />
  );
}
```

6.Nweet.js 콘솔 삭제

```javascript
const onDeleteClick = async () => {
    const ok = window.confirm("삭제하시겠습니까?");
    console.log(ok);
    if (ok) {
      console.log(nweetObj.id);
      const data = await dbService.doc(`nweets/${nweetObj.id}`).delete();
      console.log(data);
    }

```

7.Nweet.js 코드 추가

```javascript
if (nweetObj.attachmentUrl !== "")
  await storageService.refFromURL(nweetObj.attachmentUrl).delete();
```

8.Profile.js 코드 수정

```javascript
import { authService, dbService } from "fbase";
import { useState, useEffect } from "react";
import { useHistory } from "react-router-dom";

const Profile = ({ userObj, refreshUser }) => {
  const history = useHistory();
  const [newDisplayName, setNewDisplayName] = useState(userObj.displayName);

  const onLogOutClick = () => {
    authService.signOut();
    history.push("/");
  };

  const onChange = (event) => {
    const {
      target: { value },
    } = event;
    setNewDisplayName(value);
  };

  const onSubmit = async (event) => {
    event.preventDefault();
    if (userObj.displayName !== newDisplayName) {
      await userObj.updateProfile({ displayName: newDisplayName });
      refreshUser();
    }
  };

  return (
    <>
      <form onSubmit={onSubmit}>
        <input
          onChange={onChange}
          type="text"
          placeholder="Display name"
          value={newDisplayName}
        />
        <input type="submit" value="Update Profile" />
      </form>
      <button onClick={onLogOutClick}>Log Out</button>
    </>
  );
};

export default Profile;
```

# 602277120 정희헌

## 2022 05 25 13주차

1.파이어베이스 오류나서 새 프로젝트 생성하고 env 파일 새로 찍었음.

2.Home.js 코드 추가

```javascript
<input type="file" accept="image/*" />
```

3.App.js 코드 삭제

```javascript
~~<footer>&copy; { new Date().getFullYear() } Nwitter </footer>~~
```

4.Home.js 코드 추가

```javascript
const {
  target: { files },
} = event;
const theFile = files[0];
```

5.Home.js 코드 추가

```javascript
const reader = new FileReader();
reader.readAsDataURL(theFile);
```

6.Home.js 코드 추가

```javascript
reader.onloadend = (finishedEvent) => {
  const {
    currentTarget: { result },
  } = finishedEvent;
  setAttachment(result);
};
```

7.Home.js 코드 추가

```javascript
const [attachment, setAttachment] = useState("");
```

# 602277120 정희헌

## 2022 05 18 12주차

1.Home.js 코드 추가

```javascript
return (
  <>
    <from onSubmit={onSubmit}>
      <input
        value={nweet}
        onChange={onChange}
        type="text"
        placeholder="What's on your mind?"
        maxLength={120}
      />
      <input type="submit" value="Nweet" />
    </from>

    <div>
      {nweets.map((nweet) => (
        <div key={nweet.id}>
          <h4>{nweet.text}</h4>
        </div>
      ))}
    </div>
  </>
);
```

2.App.js 코드 추가 (setuserobj)

```javascript
const [userObj, setUserObj] = useState(null);
```

3.Route.js 코드 추가 (userobj)

```javascript
<Home userObj={userObj} />
            </Route>
```

3.Home.js 코드 삭제 (getNweets 안씀)

```javascript
~~ const getNweets = async () => {
    const dbNweets = await dbService.collection("nweets").get();
    dbNweets.forEach((document) => {
      const nweetObject = { ...document.data(), id: document.id };
      setNweets((prev) => [document.data(), ...prev]);
    });
  };

  getNweets(); ~~

```

3.Home.js 코드 삭제한 부분에 onSnapshot 함수 적용

```javascript
useEffect(() => {
    dbService.collection("nweets").onSnapshot((snapshot) => {
      const newArray = snapshot.docs.map((document) => ({
        id: document.id,
        ...document.data(),
      }));
      setNweets(newArray);
    });

```

4.Nweet.js 컴퍼넌트 생성

```javascript
const Nweet = ({ nweetObj }) => {
  return (
    <div>
      <h4>{nweetObj.text}</h4>
    </div>
  );
};

export default Nweet;
```

5.Home.js 코드 삭제(Nweet.js에 해당하는 부분)

```javascript
<div key={nweet.id}>
  <h4>{nweet.text}</h4>
</div>
```

6.Home.js 코드 추가

```javascript
isOwner={nweet.creatorId === userObj.uid}
```

# 602277120 정희헌

## 2022 05 11 11주차

1.fbase.js 코드 수정 (firebase dbservice 부분)

```javascript
export const dbService = firebase.firestore();
```

2.Home.js onSubmit 부분 async 추가, await 하위 코드 추가

```javascript
const onSubmit = async (event) => {
  event.preventDefault();
  await dbService.collection("nweets").add({
    text: nweet,
    createdAt: Date.now(),
  });
  setNweet("");
};
```

3.Home.js forEach, ... 코드 추가

```javascript
const Home = () => {
  const [nweet, setNweet] = useState("");
  const [nweets, setNweets] = useState([]);

  const getNweets = async () => {
    const dbNweets = await dbService.collection("nweets").get();
    dbNweets.forEach((document) => {
      const nweetObject = { ...document.data(), id: document.id };
      setNweets((prev) => [document.data(), ...prev]);
    });
  };
```

3.Home.js getNweets 추가

```javascript
const getNweets = async () => {
  const dbNweets = await dbService.collection("nweets").get();
  console.log(dbNweets);
};

useEffect(() => {
  getNweets();
}, []);
```

```javascript

```

```javascript

```

# 602277120 정희헌

## 2022 05 04 9주차

1. Auth.js google, github 로그인 코드 추가

```javascript
const {
  target: { name },
} = event;
let provider;
if (name === "google") {
  provider = new firebaseInstance.auth.GoogleAuthProvider();
} else if (name === "github") {
  provider = new firebaseInstance.auth.GithubAuthProvider();
}
```

2. Auth.js async await 추가 (로딩 딜레이 때문)

```javascript
 const onSocialClick = async (event) => { //...
 const data = await authService.signInWithPopup(provider);

```

3. Router.js 로그인 관련 코드 추가

```javascript
{
  isLoggedIn && <Navigation />;
}
```

4. Home.js 코드 수정

```javascript
import { useState } from "react";

const Home = () => {
  const [nweet, setNweet] = useState("");

  const onSubmit = (event) => {
    event.preventDefault();
  };

  const onChange = (event) => {
    event.preventDefault();
    const {
      target: { value },
    } = event;
    setNweet(value);
  };

  return (
    <from onSubmit={onSubmit}>
      <input
        value={nweet}
        onChange={onChange}
        type="text"
        placeholder="What's on your mind?"
        maxLength={120}
      />
      <input type="submit" value="Nweet" />
    </from>
  );
};

export default Home;
```

5.Navigation.js 파일 생성 및 코드 작성

```javascript
import { Link } from "react-router-dom";

const Navigation = () => {
  return (
    <nav>
      <ul>
        <li>
          <Link to="/">Home</Link>
        </li>
        <li>
          <Link to="/profile">My Profile</Link>
        </li>
      </ul>
    </nav>
  );
};

export default Navigation;
```

# 602277120 정희헌

## 2022 04 27 8주차 (7주차는 시험)

1. Auth.js async , 코드 일부 입력 누락된 것 수정 (async, await)

```javascript
const onSubmit = async (event) => {
  event.preventDefault();
  try {
    let data;
    if (newAccount) {
      // create newAccount
      data = await authService.createUserWithEmailAndPassword(email, password);
    } else {
      // log in
      data = await authService.signInWithEmailAndPassword(email, password);
    }
    console.log(data);
  } catch (error) {
    console.log(error);
  }
};
```

```javascript
const toggleAccount = () => setNewAccount((prev) => !prev);
```

2. fbase.js 부분에서 env 값 불러오는 부분 잘못 수정한거 고치기

```javascript
const firebaseConfig = {
  apiKey: process.env.REACT_APP_API_KEY,
  authDomain: process.env.REACT_APP_AUTH_DOMAIN,
  projectId: process.env.REACT_APP_PROJECT_ID,
  storageBucket: process.env.REACT_APP_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_MESSAGING_SENDER_ID,
  appId: process.env.REACT_APP_APP_ID,
};
```

3. App.js useEffect 추가, 해당 부분 코드 추가 입력 (로그인 상태 변경)

```javascript
function App() {
  const [init, setInit] = userState(false);
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  useEffect(() => {
    authService.onAuthStateChanged((user) => {
      if (user) {
        setIsLoggedIn(user);
      } else {
        setIsLoggedIn(false);
      }
      setInit(true);
    });
  }, []);

```

```javascript
{
  init ? <AppRouter isLoggedIn={isLoggedIn} /> : "initializing...";
}
```

# 602277120 정희헌

## 2022 04 13 5주차

1.App.js 코드 추가

```javascript
import { authService } from "fbase";
```

2.firebase(fbase.js) 설정, 코드 추가
구글, 이메일, 깃허브를 파이어베이스에 등록.

```javascript
// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "AIzaSyCWwAKwvWfrlS8B6xh_1eAs4R8pa1S4j_c",
  authDomain: "hhwitter-46414.firebaseapp.com",
  projectId: "hhwitter-46414",
  storageBucket: "hhwitter-46414.appspot.com",
  messagingSenderId: "881795990939",
  appId: "1:881795990939:web:46d167ad86be4c62b10d3e",
};
```

3.Auth.js 코드 추가

```javascript
const Auth = () => {
  return (
    <div>
      <form>
        <input type="email" placeholder="Email" required />
        <input type="password" placeholder="Password" required />
        <input type="submit" value="Log In" />
      </form>
      <div>
        <button>Continue with Google</button>
        <button>Continue with Github</button>
      </div>
    </div>
  );
};

export default Auth;

--------------------------

import { authService } from "fbase";
import { useState } from "react";

const Auth = () => {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [newAccount, setNewAccount] = useState(true);
  const [error, setError] = useState("");

  const onChange = (event) => {
    const {
      target: { name, value },
    } = event;
    if (name === "email") {
      setEmail(value);
    } else if (name === "password") {
      setPassword(value);
    }
  };

  const onSubmit = (event) => {
    event.preventDefault();
    if (newAccount) {
      // create newAccount
    } else {
      // log in
    }
  };

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          name="email"
          type="email"
          placeholder="Email"
          required
          value={email}
          onChange={onChange}
        />
        <input
          name="password"
          type="password"
          placeholder="Password"
          required
          value={password}
          onChange={onChange}
        />
        <input type="submit" value={newAccount ? "Create Account" : "Log In"} />
      </form>
      <div>
        <button name="google">Continue with Google</button>
        <button name="github">Continue with Github</button>
      </div>
    </div>
  );
};

export default Auth;
```

4. fbase.js 코드추가

```javascript
export const firebaseInstance = firebase;
```

# 602277120 정희헌

## 2022 04 06 4주차

1. Router.js 코드수정

```javascript
//1 ~ 4
import { useState } from "react";
import { HashRouter as Router, Route, Routes } from "react-router-dom";
import Auth from "../routes/Auth";
import HOME from "../routes/Home";
//6
const [isLoggedIn, setisLoggedIn] = useState(false);

import { HashRouter as Router, Route, Switch } from "react-router-dom";
import Auth from "routes/Auth";
import Home from "routes/Home";

const AppRouter = ({ isLoggedIn }) => {
  return (
    <Router>
      <Switch>
        {isLoggedIn ? (
          <Route exact path="/">
            <Home />
          </Route>
        ) : (
          <Route exact path="/">
            <Auth />
          </Route>
        )}
      </Switch>
    </Router>
  );
};

export default AppRouter;
```

2.App.js 코드 수정

```javascript
import { useEffect, useState } from "react";
import AppRouter from "components/Router";

function App() {
  const [isLoggedIn, setisLoggedIn] = useState(false);
  return (
    <>
      <AppRouter isLoggedIn={isLoggedIn} />
      <footer>&copy; {new Date().getFullYear()}HHwitter</footer>
    </>
  );
}

export default App;
```

3.jsconfig.json 작성

```
{
  "compilerOptions": {
    "baseUrl": "src"
  },
  "include": ["src"]
}
```

4.firebase.js => fbase.js 로 변경후 코드수정

```javascript
// Import the functions you need from the SDKs you need
import firebase from "firebase/app";
import "firebase/auth";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "REACT_APP_API_KEY",
  authDomain: "REACT_APP_AUTH_DOMAIN",
  projectId: "REACT_APP_PROJECT_ID",
  storageBucket: "REACT_APP_STORAGE_BUCKET",
  messagingSenderId: "REACT_APP_MESSAGING_SENDER_ID",
  appId: "REACT_APP_APP_ID",
};

// Initialize Firebase
firebase.initializeApp(firebaseConfig);

export const authService = firebase.auth();
```

---

# 602277120 정희헌

## 3월 30일 실무프로젝트 수업 내용 정리

1.  FireBase 프로젝트 생성

[FireBase 페이지](https://firebase.google.com/ "Firebase link")

2.  FireBase SDK 설치

- <pre><code>npm install firebase</code></pre>

3.  firebase.js 파일 생성

4.  index.js 파일에 firebase 관련 import

> Firebase 8버전 이하

```javascript
import firebase from "firebase/app";
import "firebase/auth";
import "firebase/firestore";
```

> [firebase 9버전 이상]

```javascript
import firebase from "firebase/compat/app";
import "firebase/compat/auth";
import "firebase/compat/firestore";
```

5.  .env 파일 설정

> 1.  .env 파일 생성하기
> 2.  .gitignore 파일에 .env 등록
> 3.  firebase.js 파일 각각의 값을 변수로 변경 및 변수명 앞에 process.env. 추가하여 참조

6.  프로젝트 루트에 components 및 routes 폴더 생성하기

7.  App.js 파일을 components 폴더로 이동
8.  EditProfile.js, Auth.js, Home.js, Profile.js 파일을 routes 폴더에 생성 , 파일안에 코드는 아래와 같이 추가

```javascript
const 컴포넌트 = () => <span>컴포넌트</span>;
export default 컴포넌트;
```

9.react-router-dom 설치

```
npm install react-router-dom
```

10. Router.js 파일 생성하기

```javascript
import { HashRouter as Router, Route, Switch } from "react-router-dom";

const AppRouter = () => {
  return (
    <Router>
      <Switch>
        <Route />
      </Switch>
    </Router>
  );
};
```

s
