# ⛅ Weather App


This is a simple weather app using front-end only.
To get weather for your location, there are a few different methods.
- Initially the app will auto load based on your IP. This is not accurate as it will be from your ISP and not necessarily your actual location.
- Next the app will ask for your location from your browser (a message will appear requesting consent)
- Finally you can enter a city into the search box



## Configuration



1. Create an account on [weatherAPI.com](https://www.weatherapi.com/signup.aspx)
2. Log into your new account and find your API Key which will look something like this: 4l61b368140945t99ob74346570803
3. Enter the API Key into the .env.example file for VITE_WEATHER_API_KEY and save it as .env



## Project Setup



```sh

npm install

```



### Compile and Hot-Reload for Development



```sh

npm run dev

```



### Compile and Minify for Production



```sh

npm run build

```
