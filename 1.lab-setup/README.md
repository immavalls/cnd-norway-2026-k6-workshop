# Lab setup

In this workshop, you'll learn how to create load tests to verify the performance of QuickPizza, a simple demo application, under load.

![QuickPizza screenshot](./images/quickpizza-screenshot.png)

QuickPizza is available in three environments:
- [localhost:3333](http://localhost:3333/): a local environment that runs with Docker.
- [quickpizza.grafana.com](https://quickpizza.grafana.com/): a shared test environment for small-scale load tests.
- [quickpizza.grafana.fun](https://quickpizza.grafana.fun/): a shared test environment for small-scale load tests that also lets you visualize results in [Grafana Play](https://play.grafana.org/).

## Installation steps

_**Need help?** Raise your hand and we'll come help._

Clone the repo and start QuickPizza on your local machine:

```bash
git clone https://github.com/immavalls/cnd-norway-2026-k6-workshop.git
cd cnd-norway-2026-k6-workshop
docker compose pull   # optional: the `otel-lgtm` image is over 800 MB, do this once on a good connection
docker compose up -d
```

The output should be similar to:

```bash
[+] up 7/7
✔ Network cnd-norway-2026-k6-workshop_default Created
✔ Volume cnd-norway-2026-k6-workshop_postgres_data Created
✔ Container otel-lgtm Started
✔ Container quickpizza-db Healthy
✔ Container quickpizza Started
✔ Container alloy Started
✔ Container k6 Started
```

> To reset the environment, run `docker compose down -v` to stop the containers and remove attached volumes.

QuickPizza should now be running at [localhost:3333](http://localhost:3333/). The stack also includes Grafana at [localhost:3000](http://localhost:3000/), which you'll use in later exercises.

## Check your k6 installation

You can run k6 in two ways. Pick one and use it for all the exercises.

**Option 1: k6 binary.** [Install k6 OSS](https://grafana.com/docs/k6/latest/set-up/install-k6/) with your operating system's package manager. You might need to open a new terminal session so it picks up the updated `PATH`. Then verify it:

```bash
k6 version
```

**Option 2: Docker.** No installation needed. `docker compose up -d` already started a `k6` container with this repository mounted and connected to QuickPizza and Grafana. Verify it:

```bash
docker compose exec k6 k6 version
```

The output should look similar to `k6 v2.3.0 (commit/..., go1.27.1, linux/arm64)`.

If you use Docker, run `docker compose exec k6 k6 ...` wherever the exercises show `k6 ...`. For example, `k6 run k6-test.js` becomes `docker compose exec k6 k6 run k6-test.js`. The command is the same on Linux, macOS and Windows.

Next, run a basic k6 test to verify your test environment:

```bash
k6 run k6-test.js
```

<details>
<summary>Alternative: run the test against a remote testing environment</summary>

```bash
k6 run -e BASE_URL=https://quickpizza.grafana.com k6-test.js
```

With Docker, add the prefix: `docker compose exec k6 k6 run -e BASE_URL=https://quickpizza.grafana.com k6-test.js`

</details>

The output should look similar to:

```bash
...
running (00m00.1s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [======================================] 1 VUs  00m00.1s/10m0s  1/1 iters, 1 per VU
```

You'll continue building on this k6 script in the next exercises.

## Related Resources

The following diagram shows the Docker Compose architecture used in this workshop:

![Docker Compose setup](../compose/diagram-architecture.svg)

- [k6 OSS documentation](https://grafana.com/docs/k6/latest/)
- [github.com/grafana/quickpizza](https://github.com/grafana/quickpizza): This repository can be used to learn and explore the Grafana observability stack.

---

[Workshop homepage](https://github.com/immavalls/cnd-norway-2026-k6-workshop) · [Next exercise →](../2.basic-load-test/)