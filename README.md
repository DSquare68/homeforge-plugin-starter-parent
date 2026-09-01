# Homeforge Plugin Starter Parent

Parent POM for HUB plugins. Declares the dependencies every plugin needs
(all `[provided]` except Flyway), pins the versions to whatever the HUB
host ships, and pre-configures the jar/shade build so a plugin POM only
has to say who it is and where it lives.

See `homeforge-plugin-archetype` to generate a new plugin from this parent.

## REST controllers and routes: what HUB owns vs. what you write

Your `HubPlugin` implementation can override two methods to contribute to
HUB:

- `restControllers()` — return plain objects whose methods carry normal
  Spring MVC handler annotations (`@GetMapping`, `@PostMapping`, etc.).
  Write them exactly like Spring Boot controllers, with one difference:
  **nothing is `@Autowired`** — plugins have no Spring context of their
  own, so wire whatever the controller needs (typically the `HubApi`
  passed into `onActivate`, plus your own repositories) through its
  constructor by hand.
- `routes()` — return `PluginRoute` entries (relative path + Vaadin view
  class) for your plugin's UI.

Two contracts HUB enforces, not conventions you need to remember:

1. **Your declared paths are never used verbatim.** Whatever
   `@RequestMapping`/`@GetMapping` you write, HUB rewrites the final path
   to `/api/plugins/<your plugin id>/...`. A `PluginRoute`'s `path()` is
   always relative to your own plugin path the same way. You cannot
   register outside your own namespace, and cannot collide with another
   plugin's identical path — the id prefix is what separates you.
2. **Do not declare your own Spring Security config.** There is no
   extension point for it, and there won't be one: every path under
   `/api/plugins/**` requires an authenticated HUB session, uniformly,
   with no per-plugin override. If your controller needs finer-grained
   access control, check it yourself inside the method body with
   `hubApi.user().hasRole(...)`.

## Building a plugin

```bash
mvn clean package
```

The shaded jar's `MANIFEST.MF` carries your identity (`Plugin-Id`,
`Plugin-Version`, `Hub-Name`, `Hub-Path`, `Hub-Schema`, …) from the
`plugin.*` properties in your POM — HUB reads it before your plugin
class is ever loaded.
