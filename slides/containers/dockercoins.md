# TP : Deploying a sample application

- We will connect to our new Linux lab VM

- We will clone the TP repository

.lab[

- Clone the repository on your VM:
  ```bash
  git clone https://github.com/GuillaumeMorini/TP-Docker
  ```

]

- This repository has the source code but no Dockerfile or docker-compose.yml

---

## What's this application?

--

- It is a DockerCoin miner! 💰🐳📦🚢

--

- No, you can't buy coffee with DockerCoins

--

- How DockerCoins works:

  - generate a few random bytes

  - hash these bytes

  - increment a counter (to keep track of speed)

  - repeat forever!

--

- DockerCoins is *not* a cryptocurrency

  (the only common points are "randomness", "hashing", and "coins" in the name)

---

## DockerCoins in the microservices era

- DockerCoins is made of 5 services:

  - `rng` = web service generating random bytes

  - `hasher` = web service computing hash of POSTed data

  - `worker` = background process calling `rng` and `hasher`

  - `webui` = web interface to watch progress

  - `redis` = data store (holds a counter updated by `worker`)

- These 5 services should be visible in your application's Compose file at the end of the lab.

---

## How DockerCoins works

- `worker` invokes web service `rng` to generate random bytes

- `worker` invokes web service `hasher` to hash these bytes

- `worker` does this in an infinite loop

- every second, `worker` updates `redis` to indicate how many loops were done

- `webui` queries `redis`, and computes and exposes "hashing speed" in our browser

*(See diagram on next slide!)*

---

class: pic

![Diagram showing the 5 containers of the applications](images/dockercoins-diagram.png)

---

## Service discovery in container-land

How does each service find out the address of the other ones?

--

- We do not hard-code IP addresses in the code

- We do not hard-code FQDNs in the code, either

- We just connect to a service name, and container-magic does the rest

  (And by container-magic, we mean "a crafty, dynamic, embedded DNS server")

---

## Example in `worker/worker.py`

```python
redis = Redis("`redis`")


def get_random_bytes():
    r = requests.get("http://`rng`/32")
    return r.content


def hash_bytes(data):
    r = requests.post("http://`hasher`/",
                      data=data,
                      headers={"Content-Type": "application/octet-stream"})
```

(Feel free to check the full source code of the worker!)

---

## Connecting to the web UI

- The `webui` container exposes a web dashboard; let's view it

.lab[

- Check the port allocated to the web UI:
  ```bash
  docker compose ps
  ```

- Open that in a web browser (after doing the SSH port forward)

]

A drawing area should show up, and after a few seconds, a blue
graph will appear.

---

## Bonus points

- The observed performance of dockercoins should be around 5 coins per second

- You will get bonus points if you explain how you succeed to go above 5 coins per second

- Even more bonus points if you can go above 20 coins per second

---

class: title, pic

## Up to you


![The floor is yours](images/floor_is_yours.jpg)