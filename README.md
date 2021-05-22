# spark-dashboard

## Chrome with CORS disabled
When running this app on your local machine the access of the API via Javascript might be blocked to some CORS policy issues. As I just needed as fast as possible a view on the Spark data, I did not modify the Spark API or integrate a proxy, I just started a chrome instance with CORS security features disabled. I use this dashboard for development only anyway.

You have to start Chrome with the flag `--disable-web-security`
In Windows start chrome from the programm folder `/c/Programme/Google/Chrome/Application` with
```
cd /c/Programme/Google/Chrome/Application
./chrome.exe --user-data-dir="C:/Chrome dev session" --disable-web-security
```

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

