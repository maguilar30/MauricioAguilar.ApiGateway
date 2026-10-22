# MauricioAguilar.ApiGateway

A lightweight API gateway built with ASP.NET Core and [YARP (Yet Another Reverse Proxy)](https://microsoft.github.io/reverse-proxy/). It accepts requests on the gateway and forwards matching paths to configured backend services.

## Current routes

The routes below are declared in `appsettings.json`:

| Gateway path | Backend cluster | Configured destination |
| --- | --- | --- |
| `/api/customers/{**catch-all}` | `customers-cluster` | `https://localhost:7001/` |
| `/api/order/{**catch-all}` | `orders-cluster` | `https://localhost:7002/` |
| `/api/product/{**catch-all}` | `products-cluster` | Not currently configured |

The catch-all forwards the remainder of the path to the selected backend. For example, `GET /api/customers/123` is routed to the customers service.

> **Configuration note:** The product route references `products-cluster`, but that cluster is not defined under `ReverseProxy:Clusters`. There is a destination at `https://localhost:7003/` in `appsettings.json`, but it is currently nested under a separate `orders-cluster` property rather than under `Clusters`. As a result, the product route is not configured to reach that destination. Move the destination into a `products-cluster` entry under `ReverseProxy:Clusters` to enable it.

## Requirements

- .NET 10 SDK
- The backend services running and reachable at their configured addresses
- Trusted HTTPS development certificates when using the configured HTTPS backend destinations

## Run locally

From the repository root:

```powershell
dotnet restore
dotnet run --project .\MauricioAguilar.ApiGateway.csproj
```

The launch profiles expose the gateway at `http://localhost:5239` or `https://localhost:5000` (with HTTP also available at port `5239`). The backend addresses are configured separately in `appsettings.json`.

## Try the routes

With the relevant backend services running, send requests to the gateway. For example:

```powershell
Invoke-RestMethod http://localhost:5239/api/customers/123
Invoke-RestMethod http://localhost:5239/api/order/123
```

The gateway forwards the HTTP method, path, query string, and request to the destination selected by the route configuration. A backend response requires the corresponding service to be available.

## Configuration

YARP is registered in `Program.cs` and loads route and cluster definitions from the `ReverseProxy` section in `appsettings.json`. To change a backend address, update the destination address in its cluster. To add a service, define both a route and its cluster under `ReverseProxy:Routes` and `ReverseProxy:Clusters`.

## Project structure

- `Program.cs` — configures YARP and maps reverse-proxy endpoints.
- `appsettings.json` — gateway routes, backend clusters, and logging settings.
- `Properties/launchSettings.json` — local HTTP and HTTPS launch profiles.
- `Controllers/WeatherForecastController.cs` and `WeatherForecast.cs` — template sample code; controller services and controller endpoints are currently disabled, so this is not an active gateway endpoint.
