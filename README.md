# Movify - **The Social network for Cinephiles**

Movify is a website where people can get information about their favorite movies, and share collections with their friends.
The platform is built using a powerful Redis modules like Redis Graph. It allows to perform complex queries in Cypher, while writing short code.

The app was built with FastAPI (Python), React (JS) and
uses nginx as a reverse-proxy.

![screen1](https://github.com/drhasler/redis21/raw/main/res/screen1.png)

![screen2](https://github.com/drhasler/redis21/raw/main/res/screen2.png)




## Quickstart

To launch the app locally, you will need docker-compose. For security reasons, `backend/.env` is not on this repo.
Since we rely on Google Sign-in for authentication, you will need to create a [Web client](https://console.cloud.google.com/apis/credentials?),
and register `http://127.0.0.1:8000/api/auth` as authorized redirect URI.
Generate a random string for the encryption `openssl rand -hex 16`.

Finally, create a TMDB account and [get an API key](https://developers.themoviedb.org/3/getting-started/introduction).

You can now fill the `.env` file as follows:

```
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
SECRET_KEY=
TMDB=
```

The script `run.sh` will build the images and start the containers.


## Project Structure

Most of the code is located under `frontend/` and `backend/`.

The files related to production are located under `prod/`

For development, use the `docker-compose.yaml` config file.

Interfaces are defined at `backend/main.py` and `frontend/src/actions/Actions.js`.
The interactions with Redis Graph are defined in `backend/redis_util/rg.py`.
The specialized abstract classes provide the methods upsert, delete and get/props,
which can be used without worrying about interfaces.

```python
# create classes
User = AbstractNode('User')
Follows = AbstractEdge(User, 'FOLLOWS', User)
# use
Follows.upsert('id1', 'id2', {'props': 'value'})
Follows.get('id1', None) # followed by id1
Follows.get(None, 'id2') # followers of id2
```

## Backend Database

We use the TMDB API to fetch new movies
and to search a huge collection.
Results are cached by the server with a LRU policy.

User generated data is stored in a Redis Graph graph and in JSON blobs,
where they can be updated with Redis JSON.

The graph contains 3 types of nodes (User, Movie, Collection) and 3 types of edges:

```
User -[:FOLLOWS]> User
User -[:HAS]> Collection
Collection -[:CONTAINS]> Movie {time}
```

Having collections in the Graph gives us
flexible and efficient querying.

Metadata associated to users and collections
are stored as JSON:
```
User {id, picture, name}
Collection {id, name, description}
```

### Sidenote:

Initially, we thought we could have a good use case for Redis Search,
and wrote some code in `backend/redis_util/rs.py`. The current app doesn't
use the module anymore.


## Extensibility

The website is cool, but building this app was also an opportunity
to experiment with the modules and build some tooling.

The collections our data model relies on can be transformed
to suit real world use cases: they share similarities with
roles in Identity Access Management, as they stand between users and resources.

With the AbstractNode and AbstractEdge "meta"classes, one can add a new type of
node or relation, in a single line, and get methods for CRUD operations for free,
which is quite practical and reduces the number of possible bugs.

# Getting Started with Create React App

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.\
You will also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can’t go back!**

If you aren’t satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you’re on your own.

You don’t have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn’t feel obligated to use this feature. However we understand that this tool wouldn’t be useful if you couldn’t customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: [https://facebook.github.io/create-react-app/docs/code-splitting](https://facebook.github.io/create-react-app/docs/code-splitting)

### Analyzing the Bundle Size

This section has moved here: [https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size](https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size)

### Making a Progressive Web App

This section has moved here: [https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app](https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app)

### Advanced Configuration

This section has moved here: [https://facebook.github.io/create-react-app/docs/advanced-configuration](https://facebook.github.io/create-react-app/docs/advanced-configuration)

### Deployment

This section has moved here: [https://facebook.github.io/create-react-app/docs/deployment](https://facebook.github.io/create-react-app/docs/deployment)

### `npm run build` fails to minify

This section has moved here: [https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify](https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify)
