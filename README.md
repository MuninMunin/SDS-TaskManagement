# SDS-TaskManagement

## Introduction
The project focuses on task management, offering users an improved solution for managing tasks with effective functions. With more visibility of the teamwork environment, the system allows users to work and communicate confidentially within the association. However, a system is not only about functionalities therefore Implement security into the system's design from the beginning is significant. 

## Defining User
1. **Admin ( of the System)**
    - Has the highest privileges monitoring the system.
2. **User**
    - Every user are promoted to be admin by default when they register account, and there privileges only to manage their projects that are created by them such as `assign tasks to other user` and `monitor on their project`.
        - **Functions**
            - Input Project names.
            - Input tasks.
            - Input deadline.
            - Input Status.
            - Assign task owner.
    - User that are invited to project created by other user, can only manage on tasks that are assigned to them.
        - **Functions**
            - View assigned Project and Tasks.
            - Update task status (Mark task done).

## Authentication

1. **Federate Authentication**
   ![Image](Picture/demo%20image.png)
    Beside storing user’s account information, it is also a another good choice to choose a trust third party such as google authentication for register and login and it can be built by using firebase.

    `index.js`
    ```jsx
    
    var firebaseConfig = {
        apiKey: "API-KEY",
        authDomain: "Auth-Domain",
        projectId: "ID",
        storageBucket: "ID",
        messagingSenderId: "ID",
        appId: "ID"
      };
    
      firebase.initializeApp(firebaseConfig)
      const auth = firebase.auth()
      const database = firebase.database()
      
      function register () {
        // Get all our input fields
        email = document.getElementById('email').value
        password = document.getElementById('password').value
        full_name = document.getElementById('full_name').value
    
        // Validate input
        if (validate_email(email) == false || validate_password(password) == false) {
          alert('Email or Password is empty')
          return 
        }
        if (validate_field(full_name) == false) {
          alert('One or More Extra Fields is Outta Line!!')
          return
        }
        
       
        auth.createUserWithEmailAndPassword(email, password)
        .then(function() {
          var user = auth.currentUser
          var database_ref = database.ref()
          var user_data = {
            email : email, 
            full_name : full_name,
            last_login : Date.now()
          }
      
          database_ref.child('users/' + user.uid).set(user_data)
          alert('User Created!!')
        })
        .catch(function(error) {
          var error_code = error.code
          var error_message = error.message
      
          alert(error_message)
        })
      }
      
      function login () {
        email = document.getElementById('email').value
        password = document.getElementById('password').value
      
        if (validate_email(email) == false || validate_password(password) == false) {
          alert('Email or Password is empty')
          return 
        }
      
        auth.signInWithEmailAndPassword(email, password)
        .then(function() {
          var user = auth.currentUser
          var database_ref = database.ref()
          var user_data = {
            last_login : Date.now()
          }
          database_ref.child('users/' + user.uid).update(user_data)
          alert('User Logged In!!')
        })
        .catch(function(error) {
          // Firebase will use this to alert of its errors
          var error_code = error.code
          var error_message = error.message
      
          alert(error_message)
        })
      }
    
      function validate_email(email) {
        expression = /^[^@]+@\w+(\.\w+)+\w$/
        if (expression.test(email) == true) {
          return true
        } else {
          return false
        }
      }
      
      function validate_password(password) {
        // Firebase only accepts lengths greater than 6
        if (password < 6) {
          return false
        } else {
          return true
        }
      }
      
      function validate_field(field) {
        if (field == null) {
          return false
        }
      
        if (field.length <= 0) {
          return false
        } else {
          return true
        }
      }
    
    function googleSignIn() {
      var provider = new firebase.auth.GoogleAuthProvider();
      firebase.auth().signInWithPopup(provider)
        .then(function(result) {
          var token = result.credential.accessToken;
          var user = result.user;
          var database_ref = firebase.database().ref();
          var user_data = {
            email: user.email,
            full_name: user.displayName,
            last_login: Date.now()
          };
          database_ref.child('users/' + user.uid).set(user_data);
          // User is signed in.
          alert('Google user signed in!');
        }).catch(function(error) {
          var errorCode = error.code;
          var errorMessage = error.message;
          var email = error.email;
          var credential = error.credential;
          alert(errorMessage);
        });
    }
    
    ```


2. **Password Hash**
    
    When users register their account username and password in our system, to ensure user’s data security we have implement password hash during registration this will ensure the login process only user input with the correct password are authorized. 
    
    ```jsx
    async function sha256(str) {
        const encoder = new TextEncoder();
        const data = encoder.encode(str);
        const hash = await crypto.subtle.digest('SHA-256', data);
        return Array.from(new Uint8Array(hash))
            .map(byte => ('0' + byte.toString(16)).slice(-2))
            .join('');
    }
    
    function login() {
        var username = document.getElementById("login-username").value;
        var password = document.getElementById("login-password").value;
    
        if (username.trim() === '' && password.trim() === '') {
            console.log("Please enter both username and password.");
            console.log("Login Status: Null input.\n\n");
            alert("Please enter both username and password.");
            return;
        }
    
        if (password.trim() === '') {
            console.log("Please enter password.");
            console.log("Login Status: Null input.\n\n");
            alert("Please enter password.");
            return;
        }
    
        // Hash the entered password
        sha256(password)
            .then(hashedPassword => {
                console.log("Hashed Password:", hashedPassword);
                // Read user data from the text file
                fetch('users.txt')
                    .then(response => response.text())
                    .then(data => {
                        // Split data by newline to get individual lines
                        var lines = data.split('\n');
    
                        for (var i = 0; i < lines.length; i++) {
                            var userData = lines[i].split(' : ');
    
                            if (userData[0].trim() === username) {
                                if (userData[1].trim() === hashedPassword) {
                                    console.log("Login Status:  Login successful!\n\n");
                                    document.getElementById("message").innerText = "Login successful!";
                                    return; 
                                } else {
                                    console.log("Login Status:  Wrong password.\n\n");
                                    document.getElementById("message").innerText = "Wrong password. Please try again.";
                                    return; 
                                }
                            }
                        }
    
                        // If no matching username and password found
                        document.getElementById("message").innerText = "User not exist.";
                        console.log("Login Status:  User not exist.\n\n");
                    })
                    .catch(error => {
                        console.error('Error reading the file:', error);
                        document.getElementById("message").innerText = "Error reading user data. Please try again later.";
                    });
            })
            .catch(error => {
                console.error('Error hashing the password:', error);
                document.getElementById("message").innerText = "Error hashing the password. Please try again later.";
            });
    }
    
    function register() {
        var username = document.getElementById("register-username").value;
        var password = document.getElementById("register-password").value;
    
        // Hash the entered password
        sha256(password)
            .then(hashedPassword => {
                console.log("Hashed Password:", hashedPassword);
    
                // Construct the user data string
                var userDataString = `${username}:${hashedPassword}\n`;
    
                // Read user data from the text file
                fetch('users.txt')
                    .then(response => response.text())
                    .then(data => {
                        // Check if the username already exists
                        if (data.includes(`${username}:`)) {
                            console.log("Registration Status: Username already exists.");
                            document.getElementById("register-message").innerText = "Username already exists. Please choose a different one.";
                            return;
                        }
    
                        // Append the new user data
                        data += userDataString;
    
                        // Write the updated content back to the text file
                        fetch('users.txt', {
                            method: 'POST',
                            headers: {
                                'Content-Type': 'text/plain'
                            },
                            body: data
                        })
                        .then(response => {
                            if (response.ok) {
                                console.log("Registration Status: Registration successful!");
                                document.getElementById("register-message").innerText = "Registration successful!";
                            } else {
                                console.error("Error registering user:", response.statusText);
                                document.getElementById("register-message").innerText = "Error registering user. Please try again later.";
                            }
                        })
                        .catch(error => {
                            console.error('Error registering user:', error);
                            document.getElementById("register-message").innerText = "Error registering user. Please try again later.";
                        });
                    })
                    .catch(error => {
                        console.error('Error reading the file:', error);
                        document.getElementById("register-message").innerText = "Error reading user data. Please try again later.";
                    });
            })
            .catch(error => {
                console.error('Error hashing the password:', error);
                document.getElementById("register-message").innerText = "Error hashing the password. Please try again later.";
            });
    }
    ```
