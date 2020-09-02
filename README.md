
# Virtual fitting web

> Hello world

## How to run

You should installed:

- [Node.js](https://nodejs.org)
- [Yarn](https://yarnpkg.com)

```
# Clone the repo
$ git clone https://github.com/binhmuc/virtual_fitting_web
$ cd virtual_fitting_web

# Install dependencies
$ yarn  # or npm install

# Run
$ yarn dev  # or npm run dev
```
Open your favorite browser at `http://localhost:8080`, the site is there.

## Config 

```
cd config.dev.env.js
Change to BASE_API:'"https://35.200.55.29/api/"' by another host
```
## Sample Data

Go to "src/assets" and push image with format ".jpg" in each folder below:

bot_persons: folder of human bottom try on

top_persons: folder of human top try on

bots: folder of human cloth bot try on

tops: folder of human cloth bot try on

## Acknowledgments

This is a project of **Virtual fitting ** taught by Dr. 정선태.


[Vue.js](https://vuejs.org/)

## Authors

Pham Duy Lai, Nhat Tan Nguyen

## License
Copyright © 2018, Fing

Released under the [MIT License](https://opensource.org/licenses/MIT).
