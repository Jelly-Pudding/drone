# Drone Server Installation

## Ngrok Installation
Your Drone server will need to be publicly accessible. In this example, `ngrok` will be used. Download instructions for Linux can be found [here](https://ngrok.com/download). You can also follow the below steps:

> 1. Run `wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz`.
> 2. Extract ngrok with `sudo tar xvzf your/present/directory/file -C /usr/local/bin`.
> 3. [Sign up](https://ngrok.com/) with ngrok in order to get an authtoken.
> 4. Add the authtoken with ngrok config add-authtoken token-goes-here
> 5. Start ngrok with this command: `ngrok http 80`.
> 6. You will be publicly available at the URL underlined below: 
![image](./pictures/ngrok.png)


## Drone Installation for GitHub
You can follow the instructions on [Drone's official documentation page](https://docs.drone.io/server/provider/github/) for installing a Drone server for GitHub. Alternatively, you can follow the steps below:

> 1. [Create a GitHub OAuth application](https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app). You can name the application `Drone`, and for the `Homepage URL` and the `Authorization callback URL`, put in the publicly accessible ngrok URL followed by `/login` at the end.
> 2. Make a note of the `Client ID` and especially the `Client Secret` which you won't be able to see again. 
> 3. In addition to these secrets, create a shared secret by typing this command into the terminal: `openssl rand -hex 16`. Also make a note of this secret.
> 4. Install Docker. Remember you might need to run an update and an upgrade first.
> 5. Pull the Drone docker image with this command: `docker pull drone/drone:2`. 
> 5. Execute this docker run command:
```bash
docker run \
  --volume=/var/lib/drone:/data \
  --env=DRONE_GITHUB_CLIENT_ID=your-id \
  --env=DRONE_GITHUB_CLIENT_SECRET=super-duper-secret \
  --env=DRONE_RPC_SECRET=super-duper-secret \
  --env=DRONE_SERVER_HOST=https://211f-2-121-41-213.eu.ngrok.io \
  --env=DRONE_SERVER_PROTO=https \
  --publish=80:80 \
  --publish=443:443 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:2
```
The `DRONE_RPC_SECRET` is the secret created by running the command `openssl rand -hex 16`. The server host is the publicly accessible ngrok URL.
> 6. You will be able to access the Drone server by typing the ngrok URL into your browser. 