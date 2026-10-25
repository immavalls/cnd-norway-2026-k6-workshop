# Define workload in requests per second

_**Need help?** Raise your hand and we'll come help._

Requests per second (RPS), also known as request rate, is the standard metric used to measure traffic for APIs and services.

We've observed that our QuickPizza service handles around **30 requests per second** during normal operation.

Instead of modeling the workload in terms of users, let's use the observed traffic value to generate a workload of **30 requests per second**.

For simplicity, we'll:
- Set the test duration to `2m`, long enough to watch the workload in the dashboard.
- Use the [`constant-arrival-rate`](https://grafana.com/docs/k6/latest/using-k6/scenarios/executors/constant-arrival-rate/) executor to generate a constant (no ramping) iteration rate.

Update the `k6-test.js` file at the root of the repository with the following options, and fill in the missing values:

```js
export const options = {
  scenarios: {
    default: {
      executor: 'constant-arrival-rate',

      // string: How long the test lasts
      duration: ,

      // integer: Iteration rate, the number of iterations to start during each `timeUnit` period
      rate: ,

      //string: Default "1s". Period of time to apply the `rate` value.
      timeUnit: ,

      //integer: Number of VUs to pre-allocate before test start to preserve runtime resources.
      preAllocatedVUs: 50,
    },
  },
};
```

When your `k6-test.js` is ready, run it with the `--out` option. Run the commands from the repository root.

**k6 binary**

```bash
k6 run --out web-dashboard k6-test.js
```

**Docker**

```bash
docker compose exec k6 k6 run --out web-dashboard k6-test.js
```

Open the web dashboard while the test is running:

- **k6 binary**: [http://127.0.0.1:5665/ui/](http://127.0.0.1:5665/ui/)
- **Docker**: [http://127.0.0.1:5666/ui/](http://127.0.0.1:5666/ui/)

You can verify the generated workload in the terminal output:

```bash
running (02m00.2s), 000/50 VUs, 3600 complete and 0 interrupted iterations
default ✓ [======================================] 000/50 VUs  2m0s  30.00 iters/s

...

# number of HTTP requests and HTTP request per second
http_reqs......................: 3601   29.9/s

# number of script iterations and iterations per second
iterations.....................: 3601   29.9/s
```

The exact numbers will differ slightly on your machine. On the **default ✓** row, `30.00 iters/s` indicates the iteration rate configured with the `constant-arrival-rate` executor.

⚠️ Do `http_reqs` and `iterations` report significantly lower rates than the configured `30 iters/s`?

🔍 If so, review the test scenario and find what could cause requests to execute at a slower pace.

After fixing it, run the test again.

<details>
<summary>Want to test the solution?</summary>

The solution for this exercise is in [`answer/k6-test.js`](./answer/k6-test.js). It's a separate file: it doesn't replace your `k6-test.js` at the root of the repository, and you run it by its path. Run the following command from the repository root:

**k6 binary**

```bash
k6 run --out web-dashboard 3.workload-in-rps/answer/k6-test.js
```

**Docker**

```bash
docker compose exec k6 k6 run --out web-dashboard 3.workload-in-rps/answer/k6-test.js
```

Open the dashboard while the test is running:

- **k6 binary**: [http://127.0.0.1:5665/ui/](http://127.0.0.1:5665/ui/)
- **Docker**: [http://127.0.0.1:5666/ui/](http://127.0.0.1:5666/ui/)

</details>

## Related Resources

- [k6 Documentation: Executors](https://grafana.com/docs/k6/latest/using-k6/scenarios/executors/)
- [k6 Documentation: Constant arrival rate executor](https://grafana.com/docs/k6/latest/using-k6/scenarios/executors/constant-arrival-rate/)
- [k6 Documentation: Load test types](https://grafana.com/docs/k6/latest/testing-guides/test-types/)

---

[← Previous exercise](../2.basic-load-test/) · [Workshop homepage](https://github.com/immavalls/cnd-norway-2026-k6-workshop) · [Next exercise →](../4.assertions/)