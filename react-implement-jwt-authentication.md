# JWT Authentication on React Templates

How to implement JWT Authentication to a React Template **for beginners**, step by step.

<br />

## Starting Point

To use the product, Node JS (>= 12.x) is required and GIT to clone/download the project from the public repository.

**Step #1** - Clone the given project

```bash
$ git clone https://github.com/user/project
$ cd project
```

<br >

**Step #2** - Install dependencies via NPM or yarn

```bash
$ npm i
// OR
$ yarn
```

<br />

**Step #3** - Start in development mode

```bash
$ npm run start 
// OR
$ yarn start
```

<br />

## Analyze The Codebase

**Step #1** - Look around the project and see how its content is structured

<br />

**Step #2** - Detect master pages

Often times the project is separated in 2 or 3 main sides:

- /auth
- /admin or /dashboard
- /rtl (right to left text)

<br />

**Step #3** - Detect authentication pages

Now you need to locate the authentication pages. Those are usually called **SignIn, Login, SignUp, Register**. The logout page does not always exist and you have to create it.

<br />

## Code The API and Store

**Step #1** - Create the frontend API

Start by creating the frontend API using **axios**

- in **/src** you can create a new folder called "api"
- add a **index.js** file with the api configuration:

```javascript
import Axios from "axios";
import { API_SERVER } from "../config/constant";

const axios = Axios.create({
  baseURL: `${API_SERVER}`,
  headers: { "Content-Type": "application/json" },
});

axios.interceptors.request.use(
  (config) => {
    return Promise.resolve(config);
  },
  (error) => Promise.reject(error)
);

axios.interceptors.response.use(
  (response) => Promise.resolve(response),
  (error) => {
    return Promise.reject(error);
  }
);

export default axios;
});
```  

- next, add a **auth.js** file which will contain your API calls:

```javascript
import axios from "./index";

class AuthApi {

  static Login = (data) => {
    return axios.post(`users/login`, data);
  };

  // don't forget to add the register and logout methods
}

export default AuthApi;
```  

- create the backend API server address in a new folder in the **src** directory: `src/config/constant.js`

This is the **magic line** that connects the live product to the **Node JS API Server**

```javascript
export const API_SERVER = "http://localhost:5000/api/";
```

But in the development process, you'll actually need to use `https://api-server-nodejs.appseed.us/api/` in order to be able to properly test your auth calls

There, your API is all configured.

<br />

**Step #2** - Create the app's store

In many React Apps you will find that their store is based on React Redux. Here we use React Context.
We need to create the store in order to keep track of the user's account and determine whether we should allow the user on certain pages if they are not logged in.

- in the **src** directory create a new context folder: `src/context/auth.context.js`:

```javascript
const AuthContext = React.createContext(null);

export const AuthProvider = ({ userData, children }) => {
  let [user, setUser] = React.useState(userData);

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => React.useContext(AuthContext);
```

- then we need to wrap the Context Provider around our **App** component

```javascript
const App = () => {
  // we get the user from the localStorage because that's where we will save their account on the login process
  let user = localStorage.getItem("user");
  user = JSON.parse(user);

  return (
    <AuthProvider userData={user}>
      <Routes />
    </AuthProvider>
  );
};

export default App;
```

<br />

**Step #3** - Create the protected routes

We need to create a protected route in order to not let the user visit the app if they're not authenticated.

```javascript
export const ProtectedRoute = ({ ...rest }) => {
  let { user } = useAuth();

  if (!user || !user.token || user.token === "") {
    return (
      // component which inform the user that they must be logged in
    );
  }

  // let user through if they're logged in
  return <Route {...rest} />;
};
```

Here we have 2 examples of both protected and unprotected routes:

```javascript
<ProtectedRoute path="/rtl" component={RtlLayout} />
<Route path="/auth" component={AuthLayout} />
```

<br />

## Code The Auth Flow

After you have identified the Auth Components you can start creating the Authentication process<br>We'll take the Login Component as an example:

```javascript
  const login = async (event) => {
    if (event) {
      event.preventDefault();
    }

    //handle exceptions like: no email entered, no password entered, here.

    try {
      let response = await AuthApi.Login({
        email,
        password,
      });
      if (response.data && response.data.success === false) {
        //display error coming from server
        return setError(response.data.msg);
      }
      return setProfile(response);
    } catch (err) {
      //display error originating from server / other sources
      console.log(err);
      if (err.response) {
        return setError(err.response.data.msg);
      }
      return setError("There has been an error.");
    }
  };
```
<br/>

```javascript
  const setProfile = (response) => {
    let user = { ...response.data.user };
    user.token = response.data.token;
    user = JSON.stringify(user);
    //setUser is imported from the useAuth React Context
    setUser(user);
    //also set the user in local storage
    localStorage.setItem("user", user);
    return history.push("/admin/dashboard");
  };
```

<br/>

## You're Done!
That is pretty much it! You'll have to repeat and update the process accordingly to the Register and Logout methods.

