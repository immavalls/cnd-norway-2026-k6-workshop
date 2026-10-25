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

You can run k6 in two ways. Pick one and use it for all the exercises. Run both checks below: the version check, and a first test against QuickPizza.

### Option 1: k6 binary

[Install k6 OSS](https://grafana.com/docs/k6/latest/set-up/install-k6/) with your operating system's package manager. You might need to open a new terminal session so it picks up the updated `PATH`.

1. Check the version:

   ```bash
   k6 version
   ```

2. Run the test:

   ```bash
   k6 run k6-test.js
   ```

### Option 2: Docker

No installation needed. `docker compose up -d` already started a `k6` container with this repository mounted and connected to QuickPizza and Grafana. The commands are the same on Linux, macOS and Windows.

1. Check the version:

   ```bash
   docker compose exec k6 k6 version
   ```

2. Run the test:

   ```bash
   docker compose exec k6 k6 run k6-test.js
   ```

> In all the next exercises, if you use Docker, run `docker compose exec k6 k6 ...` wherever the exercises show `k6 ...`.

### Check the output

The version check should print something similar to the following. Your version number and platform may differ:

```
k6 v2.3.0 (commit/e088784614, go1.27.1, linux/arm64)
```

The test should print a randomly generated pizza name, and finish with `1/1 iters` and no failed requests. The pizza name, timings and sizes will differ:

```
     execution: local
        script: k6-test.js
        output: -

     scenarios: (100.00%) 1 scenario, 1 max VUs, 10m30s max duration (incl. graceful stop):
              * default: 1 iterations for each of 1 VUs (maxDuration: 10m0s, gracefulStop: 30s)

time="2026-10-25T12:06:04Z" level=info msg="The Mushy Romana (5 ingredients)" source=console

  █ TOTAL RESULTS

    HTTP
    http_req_duration..............: avg=13.79ms min=13.79ms med=13.79ms max=13.79ms p(90)=13.79ms p(95)=13.79ms
    http_req_failed................: 0.00% 0 out of 1
    http_reqs......................: 1     61.495887/s
...
running (00m00.0s), 0/1 VUs, 1 complete and 0 interrupted iterations
default ✓ [ 100% ] 1 VUs  00m00.0s/10m0s  1/1 iters, 1 per VU
```

If `http_req_failed` is not `0.00%`, QuickPizza isn't reachable. Check that `docker compose ps` shows all the containers as running, then ask for help.

<details>
<summary>Alternative: run the test against a remote testing environment</summary>

If the local QuickPizza is not working, run the same checks against the shared environment. Run the version check as above, then the test:

```bash
# k6 binary
k6 run -e BASE_URL=https://quickpizza.grafana.com k6-test.js
```

```bash
# Docker
docker compose exec k6 k6 run -e BASE_URL=https://quickpizza.grafana.com k6-test.js
```

The output should look the same as above, with `1/1 iters` and `http_req_failed` at `0.00%`.

</details>

You'll continue building on this k6 script in the next exercises.

## Related Resources

The following diagram shows the Docker Compose architecture used in this workshop:

![Docker Compose setup](../compose/diagram-architecture.svg)

- [k6 OSS documentation](https://grafana.com/docs/k6/latest/)
- [github.com/grafana/quickpizza](https://github.com/grafana/quickpizza): This repository can be used to learn and explore the Grafana observability stack.

---

[Workshop homepage](https://github.com/immavalls/cnd-norway-2026-k6-workshop) · [Next exercise →](../2.basic-load-test/)