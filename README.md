# LetsVibe
A real time web-app for one to one chatting. The project is broadly divided into two parts - Server and Public (client).
<br>The Socket.io module has been used for bi-directional and low latency communication<br><br>


### Tech Stack Used
MERN <br><br>


### Hosting Platform
The Web-App (used Netlify): https://letsvibe.netlify.app/ <br>
The Server (Used Heroku): https://letsvibe-backend.herokuapp.com/ <br>

Ther Server is mainly used for handling the HTTP requests and accesing the mongoDB database hosted on mongoDB Atlas.<br><br>

## THE UI
#### Login Page
<img width="450" alt="image" src="https://user-images.githubusercontent.com/89444844/179421300-8a674c26-ebe7-4a0d-b03e-e45780bd6fd7.png"><br>
#### Registration Page
<img width="450" alt="image" src="https://user-images.githubusercontent.com/89444844/179421405-d3ef26e9-c38c-4533-ad51-0cc8075ccfab.png"><br>

#### Toasts
```react-toastify``` have been used for the purposes of notifying the users.<br><br>
<img width="450" alt="image" src="https://user-images.githubusercontent.com/89444844/179421630-78c0dccf-e0a0-4b3b-9a55-5c4e7b769022.png"><br>

#### Avatar Image
Upon Registration, the User is Prompted to set an Avatar Image out of four random avatars.<br>
These random avatars are fetched from ```https://api.multiavatar.com/4645646```<br><br>
<img width="450" alt="image" src="https://user-images.githubusercontent.com/89444844/179421785-8d6ac5ae-4f67-46c8-8806-4a5ac313ef0f.png"><br>

#### Welcome Page
<img width="450" alt="image" src="https://user-images.githubusercontent.com/89444844/179421859-1c43143e-f9be-488e-84a5-6e325b28e263.png"><br>

#### The Chat
<img width="450" alt="image" src="https://user-images.githubusercontent.com/89444844/179421930-d7bde626-3113-4f15-bc96-d8f2e5bfb194.png"> </t></t> <img width="450" alt="image" src="https://user-images.githubusercontent.com/89444844/179421960-568fe2de-f601-4a4e-9dd4-3ce86b07b264.png"><br>
```import Picker from "emoji-picker-react"``` for implementing the emoji's Input.<br><br>

## The App and Server
### Mongoose Models
Two Mongoose Models have been used - ```userModel``` & ```messageModel``` <br>
- The ```userModel``` stores the basic user details like ```username``` ```email``` ```password``` ```isAvatarImageSet``` ```avatarImage``` <br>
- The ```messageModel``` contains the ```message``` ```users``` ```sender``` <br>
#### The userSchema
```js
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    min: 3,
    max: 20,
    unique: true,
  },
  email: {
    type: String,
    required: true,
    unique: true,
    max: 50,
  },
  password: {
    type: String,
    required: true,
    min: 8,
  },
  isAvatarImageSet: {
    type: Boolean,
    default: false,
  },
  avatarImage: {
    type: String,
    default: "",
  },
});
```

#### The messageSchema
```js
const MessageSchema = mongoose.Schema(
  {
    message: {
      text: { type: String, required: true },
    },
    users: Array,
    sender: {
      type: mongoose.Schema.Types.ObjectId,
      ref: "User",
      required: true,
    },
  },
  {
    timestamps: true,
  }
);
``` 
 <br>
 
### User Registration and Login
The input changes in the login/regsitartion forms are continuously tracked using ```{useState} from "react" ```
On submitting, the ```handleSubmit``` functions for the respective action (login/register) are called using the ```onSubmit``` attribute.
```js
  <form action="" onSubmit={(event) => handleSubmit(event)}>
```
- ```handleSubmit``` for Login <br><br>
  ```js
  const handleSubmit = async (event) => {
    event.preventDefault();
    if (validateForm()) {
      const { username, password } = values;
      const { data } = await axios.post(loginRoute, {
        username,
        password,
      });
      if (data.status === false) {
        toast.error(data.msg, toastOptions);
      }
      if (data.status === true) {
        localStorage.setItem(
          process.env.REACT_APP_LOCALHOST_KEY,
          JSON.stringify(data.user)
        );
        navigate("/");
      }
    }
  };
  ```
- ```handleSubmit``` for Registration <br><br>
  ```js
  const handleSubmit = async (event) => {
    event.preventDefault();
    if (handleValidation()) {
      const { email, username, password } = values;
      const { data } = await axios.post(registerRoute, {
        username,
        email,
        password,
      });
      // console.log(data);
      if (data.status === false) {
        toast.error(data.msg, toastOptions);
      }
      if (data.status === true) {
        localStorage.setItem(
          process.env.REACT_APP_LOCALHOST_KEY,
          JSON.stringify(data.user)
        );
        navigate("/");
      }
    }
  };
  ```
```axios.post()``` is used for the HTTP post request through which the data is passed on to the backend server where the login and regsiter APIs are implemented.<br>
The backend server uses ```bcrypt``` which hashes and salts the password, before we store it in our database.<br>
- Register API <br>
  ```js
    module.exports.register = async (req, res, next) => {
      try {
        const { username, email, password } = req.body;
        const usernameCheck = await User.findOne({ username });
        if (usernameCheck)
          return res.json({ msg: "Username already used", status: false });
        const emailCheck = await User.findOne({ email });
        if (emailCheck)
          return res.json({ msg: "Email already used", status: false });
        const hashedPassword = await bcrypt.hash(password, 10);
        const user = await User.create({
          email,
          username,
          password: hashedPassword,
        });
        delete user.password;
        return res.json({ status: true, user });
      } catch (ex) {
        next(ex);
      }
    };
  ```
  - ```req.body``` is destructured into ```username``` ```email``` ```password```. 
  - First we check if the user already exists in our database, if so we toast an error with a message ```Username already used``` or ```Email already used``` depending on the scenario.
  - If they don't, the password (user input) undergoes hashing and ten rounds of salting. Hashing and salting exponentially increases the security level, and makes it impossible to decrypt this encrypted password.
  - This ```hashed password``` is now stored in our database alongside ```username``` and ```email```.
  
- Login API <br>
  ```js
    module.exports.login = async (req, res, next) => {
      try {
        const { username, password } = req.body;
        const user = await User.findOne({ username });
        if (!user)
          return res.json({ msg: "Incorrect Username or Password", status: false });
        const isPasswordValid = await bcrypt.compare(password, user.password);
        if (!isPasswordValid)
          return res.json({ msg: "Incorrect Username or Password", status: false });
        delete user.password;
        return res.json({ status: true, user });
      } catch (ex) {
        next(ex);
      }
    };
  ```
  - ```req.body``` is destructured into ```username``` ```password```. 
  - First we find the username in our database, if no user found, toast an error with a message ```Incorrect Username or Password```.
  - If found, we compare the user input password with the hashed password stored against the input username using ```bcrypt.compare()``` which return a boolean.
  - If evaluated to false, toast an error with the message ```Incorrect Username or Password ```.
  - Else return an object to the client, with status as true, and other user details which have been fetched from the database.
  - On the Client side, this user data is set in the ```localStorage``` for checking if the user is logged in or not using ```useEffect``` from ```react```.
  
<br>
  
### Sending Message
  
Implemented using the  ```scket.io``` model <br>
```js
  global.onlineUsers = new Map();
  io.on("connection", (socket) => {
    global.chatSocket = socket;
    socket.on("add-user", (userId) => {
      onlineUsers.set(userId, socket.id);
    });

    socket.on("send-msg", (data) => {
      const sendUserSocket = onlineUsers.get(data.to);
      if (sendUserSocket) {
        socket.to(sendUserSocket).emit("msg-recieve", data.msg);
      }
    });
  });
```
we create a ```socket``` for each user and map to each userID.  Whenever a new chat is selected and a message is send, it travels through the circuit, using the socket which has been mapped to the userID. <br>
- ```handleSendMsg``` function <br>
  ```js
    const handleSendMsg = async (msg) => {
      const data = await JSON.parse(
        localStorage.getItem(process.env.REACT_APP_LOCALHOST_KEY)
      );
      socket.current.emit("send-msg", {
        to: currentChat._id,
        from: data._id,
        msg,
      });
      await axios.post(sendMessageRoute, {
        from: data._id,
        to: currentChat._id,
        message: msg,
      });

      const msgs = [...messages];
      msgs.push({ fromSelf: true, message: msg });
      setMessages(msgs);
    };
  ```
  
  - Here ```socket.current``` is the socket associated with a particular userID.
  - ```socket.emit()``` is used for sending message to all user connected to that socket except self. Here ```socket.current``` maps to only one, thus the message is only sent to the current chat selected.
  - Also we store all the messages using our messageModel, so that they can be loaded onto to the client side in the future. Aparts, to succesfully access the correct messages in the future, we also need to store the ```from: data._id``` and ```to: currentChat._id```

<br><br>

> Configure the ```.env``` file as required and use ```npm install``` to install the required dependencies to run this web-app on your local system.<br>
> Suggestions & Improvements are welcome!
