## CORS

本节仅在跨域场景中需要（你有多个机器人 API 运行在 `localhost:8081`、`localhost:8082` 等端口，并希望将它们合并到一个 FreqUI 实例中）。

用户可以通过 `CORS_origins` 配置设置允许来自不同源 URL 的访问。

假设你的应用部署在 `https://frequi.freqtrade.io/home/`，需要以下配置：

```jsonc
{
    "jwt_secret_key": "somethingRandomSomethingRandom123",
    "CORS_origins": ["https://frequi.freqtrade.io"],
}
```

常见情况：FreqUI 可通过 `http://localhost:8080/trade` 访问。正确的配置是 `http://localhost:8080`。

!!! Tip "尾部斜杠"
    `CORS_origins` 配置中不允许尾部斜杠（例如 `"http://localhots:8080/"`），此类配置不会生效。

!!! Note
    我们强烈建议也将 `jwt_secret_key` 设置为某种随机且只有你自己知道的值。