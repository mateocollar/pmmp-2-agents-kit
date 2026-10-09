# AGENTS.md — PocketMine-MP 2.0.0 (constitution for AI agents)

> Master version lives in Spanish at [es/AGENTS.md](../es/AGENTS.md).
> Every content change applies to both files: if you edit one, edit the other.

## 1. Stack context

This document is the constitution that governs all code generated for this project.
The target stack is fixed, sacred and non-negotiable:

| Component | Mandatory version |
| --------- | ----------------- |
| PocketMine-MP | 2.0.0 (API `2.0.0`) |
| Minecraft Pocket Edition | 0.15.10 (protocol 84) |
| PHP | 7.0.14 exactly |
| Reference era | 2016 |
| Plugin format | `plugin.yml` + era-authentic phar / DevTools |

### 1.1 Mandatory warning to the agent

Your default knowledge is biased towards modern PocketMine (PM3, PM4, PM5) and
PHP 7.2-8.x, because those are the versions with the most trained examples.
**Everything that follows prevails over your trained knowledge.** If one of your API
memories contradicts this document, this document wins. If you hesitate between "what I
remember" and "what the constitution says", write what the constitution says.

Typical signs that you are wrong: you used `pocketmine\player\Player`, `World`,
`getWorld()`, string item IDs, `: void`, `?->`, `match`, `fn()`, `str_contains()` or
anything else that compiles on PHP 8 but not on 7.0.14. Review and rewrite before
delivering.

### 1.2 How to use this document

1. Read section 2 before writing a single line of PHP: the PHP 7.0 restrictions are
   what breaks this stack most often.
2. Read section 3 before using any PocketMine class: the namespaces and signatures of
   the 2.0.0 era are not the ones found in modern tutorials.
3. Follow the rules in section 4 (architecture) and section 5 (agent behavior).
4. Before delivering, walk item by item through the checklist in section 6
   (SELF-CHECK). If an item fails, do not deliver: fix it.

## 2. PHP 7.0.14 restrictions

### 2.1 General rule

The server runs on PHP 7.0.14 with the era's pthreads extension. Any syntax introduced
in 7.1 or later produces a fatal parse error when loading the plugin and the server
simply will not start it. There is no "it works anyway": either it compiles on 7.0.14
or it does not exist.

### 2.2 PROHIBITED and ALLOWED table

| Syntax or feature | Since | Status | Valid alternative on PHP 7.0 |
| ----------------- | ----- | ------ | ---------------------------- |
| `void` (return type or hint) | 7.1 | PROHIBITED | omit the type and document it with `@return void` |
| `?int`, `?string` (nullable types) | 7.1 | PROHIBITED | no type and a `@param type\|null` docblock |
| `iterable` | 7.1 | PROHIBITED | `array` or no type at all |
| `catch (A \| B $e)` (multi-catch) | 7.1 | PROHIBITED | two chained `catch` blocks |
| `[$a, $b] = $arr` (short destructuring) | 7.1 | PROHIBITED | `list($a, $b) = $arr` |
| `list("key" => $v) = $arr` | 7.1 | PROHIBITED | manual assignment `$v = $arr["key"];` |
| `public const X` (constant visibility) | 7.1 | PROHIBITED | `const X` |
| `$text[-1]` (negative offset) | 7.1 | PROHIBITED | `substr($text, -1)` |
| `object` (type) | 7.2 | PROHIBITED | no type and a docblock |
| `array_key_first()` / `array_key_last()` | 7.3 | PROHIBITED | `reset($arr)` and `end($arr)` with `key()` |
| flexible heredoc (indented closing marker) | 7.3 | PROHIBITED | closing marker at the beginning of the line |
| trailing comma in calls `f($a, $b,)` | 7.3 | PROHIBITED | drop the comma (allowed in arrays) |
| typed properties `private int $x` | 7.4 | PROHIBITED | `private $x;` with an `@var` docblock |
| arrow functions `fn($x) => ...` | 7.4 | PROHIBITED | `function ($x) { ... }` |
| coalescing assignment `??=` | 7.4 | PROHIBITED | `if (!isset($x)) { $x = ...; }` |
| spread in arrays `[...$a]` | 7.4 | PROHIBITED | `array_merge($a, $b)` |
| `__serialize()` / `__unserialize()` | 7.4 | PROHIBITED | plain properties with `serialize()` |
| `match` | 8.0 | PROHIBITED | `switch` |
| named arguments `f(cond: $x)` | 8.0 | PROHIBITED | positional arguments |
| constructor property promotion | 8.0 | PROHIBITED | declare the property and assign it in the body |
| attributes `#[Something]` | 8.0 | PROHIBITED | docblocks (`@param`, `@priority`) |
| union types `int\|string` | 8.0 | PROHIBITED | no type and validate with `is_int()` / `is_string()` |
| `mixed` / `never` | 8.0 / 8.1 | PROHIBITED | no type and a docblock |
| `str_contains()` / `str_starts_with()` / `str_ends_with()` | 8.0 | PROHIBITED | `strpos($s, $b) !== false` |
| null safe operator `?->` | 8.0 | PROHIBITED | explicit check `if ($x !== null)` |
| `throw` as an expression | 8.0 | PROHIBITED | `throw` inside its own statement |
| `get_debug_type()` / `array_is_list()` | 8.0 / 8.1 | PROHIBITED | `is_*()` and `gettype()` |
| `enum` | 8.1 | PROHIBITED | class with `const` |
| `readonly` | 8.1 | PROHIBITED | private property + getter |
| scalar type hints `int`, `string`, `bool`, `float` | 7.0 | ALLOWED | — |
| basic return types `int`, `string`, `bool`, `float`, `array`, class name | 7.0 | ALLOWED | — |
| null coalescing operator `??` | 7.0 | ALLOWED | — |
| spaceship operator `<=>` | 7.0 | ALLOWED | — |
| anonymous classes | 7.0 | ALLOWED | — |
| variadics `...$args` | 7.0 | ALLOWED | — |
| classic closures `function () use ($x) { ... }` | 7.0 | ALLOWED | — |
| `yield from`, `**`, `random_bytes()`, `intdiv()` | 7.0 | ALLOWED | — |

### 2.3 What you are allowed to use in PHP 7.0.14

PHP 7.0 already ships enough to write plugins comfortably. Incorrect example, not
compilable as-is on 7.0.14:

```php
<?php

namespace mikitoplugin\util;

class Comparador
{
    /**
     * @param string $a
     * @param string $b
     * @return int
     */
    public function comparar(string $a, string $b): int
    {
        return $a <=> $b;
    }

    /**
     * @param callable $fn
     * @param mixed ...$valores
     * @return mixed
     */
    public function aplicar(callable $fn, ...$valores)
    {
        return $fn(...$valores);
    }

    public function porDefecto($valor)
    {
        return $valor ?? "sin valor";
    }
}
```

### 2.4 Symptoms of modern code (snippets)

If your output contains any of these clues, you are writing the wrong PHP. Rewrite
before delivering. Snippets marked PROHIBITED are illustrative and deliberately do NOT
compile on 7.0.14; snippets marked CORRECT do compile.

Pair 1 — the `void` return type, the most frequent mistake in PocketMine tasks:

```php
// PROHIBITED — PHP 7.1+
public function onRun($currentTick): void
{
}

// CORRECT — PHP 7.0.14
public function onRun($currentTick)
{
}
```

Pair 2 — nullable types:

```php
// PROHIBITED — PHP 7.1+
public function buscar(?string $nombre): ?string
{
    return $nombre;
}

// CORRECT — PHP 7.0.14
/**
 * @param string|null $nombre
 * @return string|null
 */
public function buscar($nombre)
{
    return $nombre;
}
```

Pair 3 — null safe operator:

```php
// PROHIBITED — PHP 8.0
$nick = $jugador?->getDisplayName();

// CORRECT — PHP 7.0.14
$nick = "Jugador";
if ($jugador !== null) {
    $nick = $jugador->getDisplayName();
}
```

Pair 4 — `match` versus `switch`:

```php
// PROHIBITED — PHP 8.0
$etiqueta = match ($modo) {
    0 => "Supervivencia",
    1 => "Creativo",
    default => "Otro",
};

// CORRECT — PHP 7.0.14
switch ($modo) {
    case 0:
        $etiqueta = "Supervivencia";
        break;
    case 1:
        $etiqueta = "Creativo";
        break;
    default:
        $etiqueta = "Otro";
        break;
}
```

Pair 5 — arrow function versus closure:

```php
// PROHIBITED — PHP 7.4
$online = array_filter($todos, fn($p) => $p->isOnline());

// CORRECT — PHP 7.0.14
$online = array_filter($todos, function ($p) {
    return $p->isOnline();
});
```

Pair 6 — PHP 8 string helpers:

```php
// PROHIBITED — PHP 8.0
if (str_contains($nombre, "admin")) {
    $esAdmin = true;
}

// CORRECT — PHP 7.0.14
if (strpos($nombre, "admin") !== false) {
    $esAdmin = true;
}
```

Pair 7 — constructor promotion versus explicit properties:

```php
// PROHIBITED — PHP 8.0
class Entrada
{
    public function __construct(
        public string $jugador,
        public int $puntos = 0
    ) {
    }
}

// CORRECT — PHP 7.0.14
class Entrada
{
    /** @var string */
    private $jugador;

    /** @var int */
    private $puntos;

    /**
     * @param string $jugador
     * @param int $puntos
     */
    public function __construct($jugador, $puntos = 0)
    {
        $this->jugador = $jugador;
        $this->puntos = $puntos;
    }

    /**
     * @return string
     */
    public function getJugador()
    {
        return $this->jugador;
    }
}
```

Pair 8 — destructuring and multi-catch:

```php
// PROHIBITED — PHP 7.1+
[$x, $y] = $coordenadas;
try {
    guardar($datos);
} catch (RuntimeException | InvalidArgumentException $e) {
    registrar($e->getMessage());
}

// CORRECT — PHP 7.0.14
list($x, $y) = $coordenadas;
try {
    guardar($datos);
} catch (RuntimeException $e) {
    registrar($e->getMessage());
} catch (InvalidArgumentException $e) {
    registrar($e->getMessage());
}
```

Pair 9 — typed properties, `??=` and array spread:

```php
// PROHIBITED — PHP 7.4+
class Contador
{
    private int $valor = 0;
}
$config["idioma"] ??= "es";
$todo = [...$base, ...$extra];

// CORRECT — PHP 7.0.14
class Contador
{
    /** @var int */
    private $valor = 0;
}
if (!isset($config["idioma"])) {
    $config["idioma"] = "es";
}
$todo = array_merge($base, $extra);
```

Pair 10 — enum versus constants class:

```php
// PROHIBITED — PHP 8.1
enum Prioridad
{
    case Baja;
    case Alta;
}

// CORRECT — PHP 7.0.14
class Prioridad
{
    const BAJA = 0;
    const ALTA = 1;
}
```

Pair 11 — visible constant and negative offset:

```php
// PROHIBITED — PHP 7.1+
public const MAXIMO = 10;
$ultimo = $nombre[-1];

// CORRECT — PHP 7.0.14
const MAXIMO = 10;
$ultimo = substr($nombre, -1);
```

### 2.5 declare(strict_types=1)

`declare(strict_types=1)` is **plugin-optional**: it is not required and the stack does
not demand it. If you use it, respect these two rules:

1. It must be the first statement of the file, before `namespace` and any `use`.
2. Apply it consistently across the whole plugin: do not mix files with and without
   strict_types in the same codebase.

Practical warning from the era: many PocketMine 2.0.0 APIs return loose values (for
example integers coming from YAML or from the network). With strict_types, a strict
comparison or assignment can throw a `TypeError` in production. If unsure, leave it
out and validate types by hand with `is_int()`, `is_numeric()` and explicit `(int)` /
`(string)` casts.

## 3. PocketMine-MP 2.0.0 restrictions

### 3.1 Flat namespaces (the golden rule)

In the 2.0.0 API, class names live in flat, lowercase-folder namespaces.
**They do not exist**: `pocketmine\player\Player`, `pocketmine\world\World` or
`pocketmine\server\Server` — those are renames from modern versions.

| Area | Correct namespace on PM 2.0.0 |
| ---- | ----------------------------- |
| Player and server | `pocketmine\Player`, `pocketmine\Server` |
| Worlds (levels) | `pocketmine\level\Level`, `pocketmine\level\Position`, `pocketmine\level\Location` |
| Items and blocks | `pocketmine\item\Item`, `pocketmine\block\Block` |
| NBT | `pocketmine\nbt\tag\CompoundTag`, `pocketmine\nbt\tag\StringTag`, `pocketmine\nbt\tag\IntTag` |
| Plugins | `pocketmine\plugin\PluginBase`, `pocketmine\plugin\PluginManager` |
| Commands | `pocketmine\command\Command`, `pocketmine\command\CommandSender` |
| Events | `pocketmine\event\Listener`, `pocketmine\event\player\*`, `pocketmine\event\block\*` |
| Scheduler | `pocketmine\scheduler\Task`, `pocketmine\scheduler\TaskHandler` |
| Utilities | `pocketmine\utils\Config`, `pocketmine\utils\TextFormat` |
| Math | `pocketmine\math\Vector3` |

Derived rules:

1. The world is called `Level`, never `World`. `$player->getLevel()`, never
   `$player->getWorld()`.
2. `TextFormat` lives in `pocketmine\utils\TextFormat` and its constants are
   `TextFormat::GREEN`, `TextFormat::RED`, `TextFormat::BOLD`... with no `FORMAT_`
   prefixes and no modern static helpers.
3. Items and blocks are referenced by numeric ID from the 0.15 era. String IDs such as
   `"minecraft:stone"` do not exist yet.

### 3.2 PM5 to PM2 mapping table

If you "remember" a modern PocketMine API, write the right-hand column:

| If your memory says (PM3/PM4/PM5) | Write this on PM 2.0.0 |
| --------------------------------- | ---------------------- |
| `pocketmine\player\Player` | `pocketmine\Player` |
| `pocketmine\server\Server` | `pocketmine\Server` |
| `pocketmine\world\World` | `pocketmine\level\Level` |
| `pocketmine\world\Position` | `pocketmine\level\Position` |
| `pocketmine\world\Location` | `pocketmine\level\Location` |
| `$player->getWorld()` | `$player->getLevel()` |
| `$world->getBlock(...)` | `$level->getBlock(...)` |
| `VanillaItems::DIAMOND_SWORD()` | `Item::get(276)` |
| `VanillaBlocks::STONE()` | `Block::get(1)` |
| `ItemTypeIds::STONE` / `"minecraft:stone"` | `1` (numeric ID) |
| `$player->getGamemode()` returns a `GameMode` enum | `$player->getGamemode()` returns `int` (0, 1, 2) |
| `GameMode::SURVIVAL()` | `0` (`Player::SURVIVAL` [VERIFICAR]) |
| `Task::onRun()` with no arguments | `Task::onRun($currentTick)` |
| `plugin.yml` with `api: [5.0.0]` | `plugin.yml` with `api: [2.0.0]` |
| `pocketmine\event\player\PlayerJoinEvent` | same, no change |
| `pocketmine\utils\TextFormat` | same, no change |

### 3.3 Era-authentic plugin.yml

Every plugin ships a complete `plugin.yml` at the plugin root. Minimal correct example
from the era:

```yaml
name: MiPlugin
version: 1.0.0
main: mikitoplugin\Main
api: [2.0.0]
author: YourName
description: Example plugin for PocketMine-MP 2.0.0
commands:
  puntos:
    description: Shows your points
    usage: "/puntos"
    permission: mikitoplugin.puntos
permissions:
  mikitoplugin.puntos:
    description: Allows using the /puntos command
    default: true
```

Era notes:

1. `api` is declared as an era list (`api: [2.0.0]`); never mix in modern API versions.
2. `main` is the fully qualified class name with `\` separators, without double quotes
   (in YAML double quotes would turn `\` into an escape sequence).
3. The `main` value points at the class whose file lives under `src/` following PSR-4
   when using DevTools in source mode; in a compiled phar the classes sit at the root.
   Check which variant your project uses.
4. Do not add modern plugin.yml fields (`loadbefore`, `creators`...) that the era's
   parser does not know about.

### 3.4 Lifecycle and plugin structure

The main class extends `PluginBase` and usually implements `Listener` as well. Full
example compilable on PHP 7.0.14:

```php
<?php

namespace mikitoplugin;

use pocketmine\event\Listener;
use pocketmine\event\player\PlayerJoinEvent;
use pocketmine\plugin\PluginBase;
use pocketmine\utils\TextFormat;

class Main extends PluginBase implements Listener
{
    /** @var array */
    private $puntos = [];

    public function onEnable()
    {
        $this->getServer()->getPluginManager()->registerEvents($this, $this);
        // PluginBase does not expose getScheduler()
        // $this->getServer()->getScheduler()->scheduleRepeatingTask(new task\ResumenTask($this), 1200);
    }

    public function onDisable()
    {
        $this->puntos = [];
    }

    /**
     * @priority NORMAL
     * @ignoreCancelled true
     */
    public function onJoin(PlayerJoinEvent $event)
    {
        $nombre = strtolower($event->getPlayer()->getName());
        if (!isset($this->puntos[$nombre])) {
            $this->puntos[$nombre] = 0;
        }
        $event->getPlayer()->sendMessage(
            TextFormat::GREEN . "Welcome, " . TextFormat::WHITE . $event->getPlayer()->getName()
        );
    }

    /**
     * @param string $nombre
     * @param int $cantidad
     * @return int
     */
    public function sumarPuntos($nombre, $cantidad)
    {
        $clave = strtolower($nombre);
        if (!isset($this->puntos[$clave])) {
            $this->puntos[$clave] = 0;
        }
        $this->puntos[$clave] += $cantidad;
        return $this->puntos[$clave];
    }
}
```

Lifecycle rules:

1. `onEnable()` is the single entry point: events, commands and tasks are registered
   there, and configuration is loaded there.
2. The `PluginBase` constructor is not overridden with logic: only trivial property
   initialization if needed. Never touch the server from a constructor.
3. `onDisable()` releases state and cancels pending tasks.
4. Lifecycle methods are declared without a return type (PHP 7.0 has no `void`).

### 3.5 Commands

Commands of the era are classes extending `pocketmine\command\Command` that implement
`execute()`. Compilable example:

```php
<?php

namespace mikitoplugin\command;

use mikitoplugin\Main;
use pocketmine\command\Command;
use pocketmine\command\CommandSender;
use pocketmine\utils\TextFormat;

class PuntosCommand extends Command
{
    /** @var Main */
    private $plugin;

    /**
     * @param Main $plugin
     */
    public function __construct(Main $plugin)
    {
        parent::__construct("puntos", "Shows your points", "/puntos", ["pt"]);
        $this->setPermission("mikitoplugin.puntos");
        $this->plugin = $plugin;
    }

    /**
     * @param CommandSender $sender
     * @param string $label
     * @param array $args
     * @return void
     */
    public function execute(CommandSender $sender, $label, array $args)
    {
        if (!$this->testPermission($sender)) {
            return;
        }
        if (count($args) > 1) {
            $sender->sendMessage(TextFormat::RED . "Usage: /puntos [player]");
            return;
        }
        $nombre = count($args) === 1 ? $args[0] : $sender->getName();
        $total = $this->plugin->sumarPuntos($nombre, 0);
        $sender->sendMessage(TextFormat::AQUA . $nombre . TextFormat::WHITE . " has " . $total . " points");
    }
}
```

Registration inside `Main::onEnable()`:

```php
$this->getServer()->getCommandMap()->register("mikitoplugin", new PuntosCommand($this));
```

Notes:

1. `setPermission()` and `testPermission()` follow the era's Bukkit style.
2. You can also declare the command in `plugin.yml` under `commands:`; using
   `CommandExecute`, handling is delegated to your command class.
3. `execute()` has no return type and its body always validates `$args` and
   permissions before doing anything.

### 3.6 Events

Listeners implement `pocketmine\event\Listener`. Each handler is a public method with
a single type-hinted event parameter and metadata in the docblock:

```php
/**
 * @priority NORMAL
 * @ignoreCancelled true
 */
public function onJoin(PlayerJoinEvent $event)
{
    // ...
}
```

Rules:

1. Registration happens once in `onEnable()` with
   `$this->getServer()->getPluginManager()->registerEvents($this, $this);`.
2. `@priority` accepts `LOWEST`, `LOW`, `NORMAL`, `HIGH`, `HIGHEST` and `MONITOR`.
3. `@ignoreCancelled` accepts `true` or `false`; if the event can be cancelled and you
   care about the outcome, use `true`.
4. Handlers without a priority docblock run with the era's default priority; always
   state the intent explicitly.
5. Do no heavy work inside high-volume handlers (`PlayerMoveEvent` fires dozens of
   times per second): delegate to a scheduler task.

Most used events of the era:

| Event | Namespace | Useful methods |
| ----- | --------- | -------------- |
| `PlayerJoinEvent` | `pocketmine\event\player` | `getPlayer()`, `setJoinMessage()` |
| `PlayerQuitEvent` | `pocketmine\event\player` | `getPlayer()`, `setQuitMessage()` |
| `PlayerChatEvent` | `pocketmine\event\player` | `getPlayer()`, `getMessage()`, `setMessage()`, `setFormat()` |
| `PlayerMoveEvent` | `pocketmine\event\player` | `getPlayer()`, `getFrom()`, `getTo()` |
| `PlayerCommandPreprocessEvent` | `pocketmine\event\player` | `getPlayer()`, `getMessage()`, `setCancelled()` |
| `PlayerInteractEvent` | `pocketmine\event\player` | `getPlayer()`, `getBlock()`, `getItem()`, `getAction()` |
| `PlayerDeathEvent` | `pocketmine\event\player` | `getPlayer()`, `getEntity()` |
| `BlockBreakEvent` | `pocketmine\event\block` | `getPlayer()`, `getBlock()`, `setCancelled()` |
| `BlockPlaceEvent` | `pocketmine\event\block` | `getPlayer()`, `getBlock()`, `setCancelled()` |
| `EntityDamageEvent` | `pocketmine\event\entity` | `getEntity()`, `setDamage()`, `setCancelled()` |

`setCancelled()` / `isCancelled()` exist on the era's cancellable events; cancelled
events must not mutate server state.

### 3.7 Scheduler and tasks

Periodic tasks are classes extending `pocketmine\scheduler\Task` that override
`onRun()` **without a return type** (remember: `void` does not exist in PHP 7.0):

```php
<?php

namespace mikitoplugin\task;

use mikitoplugin\Main;
use pocketmine\scheduler\Task;

class ResumenTask extends Task
{
    /** @var Main */
    private $plugin;

    /**
     * @param Main $plugin
     */
    public function __construct(Main $plugin)
    {
        $this->plugin = $plugin;
    }

    /**
     * @param int $currentTick
     * @return void
     */
    public function onRun($currentTick)
    {
        $this->plugin->getLogger()->info("Summary ran on tick " . $currentTick);
    }
}
```

Era scheduler API:

| Operation | Call |
| --------- | ---- |
| Repeat every N ticks | `$scheduler->scheduleRepeatingTask($task, 20)` |
| Run after N ticks | `$scheduler->scheduleDelayedTask($task, 100)` |
| Cancel | `$handler->cancel();` on the returned `TaskHandler` [VERIFICAR] |

Notes:

1. 20 ticks equal one second; anything that "feels like time" is measured in ticks.
2. `onRun($currentTick)` receives the current tick; do not add a return type or extra
   parameters.
3. Never use `sleep()`, long loops or blocking `file_get_contents()` inside `onRun()`
   or inside a listener: you freeze the whole server's tick loop.
4. `AsyncTask` and the pthreads madness do exist in the era, but for simple plugins
   they complicate more than they help: v1 rule, synchronous tasks only.
5. Keep the `TaskHandler` if you need to cancel the task in `onDisable()`.

### 3.8 Config and TextFormat

`pocketmine\utils\Config` is the standard store for configuration and small state:

```php
<?php

namespace mikitoplugin;

use pocketmine\utils\Config;

// inside onEnable()
$configFile = $this->getDataFolder() . "config.yml";
$config = new Config($configFile, Config::YAML, [
    "intervalo" => 20,
    "mensaje" => "Bienvenido",
]);
$intervalo = $config->get("intervalo", 20);
if (!is_int($intervalo)) {
    $intervalo = 20;
}
$config->set("ultimoUso", time());
$config->save();
```

Notes:

1. The era's `Config` types are `Config::YAML` as the main use, plus serialized
   variants of the class itself.
2. `Config::get($key, $default)` returns whatever you stored with no type guarantees:
   always validate with `is_int()`, `is_numeric()` or `is_string()` before using it in
   calculations.
3. The file lives in `$this->getDataFolder()`; never write outside that folder.

Colors and formatting with `pocketmine\utils\TextFormat`:

```php
use pocketmine\utils\TextFormat;

$player->sendMessage(TextFormat::GREEN . "Points: " . TextFormat::WHITE . $puntos);
$clean = TextFormat::clean($userMessage); // This method DOES NOT EXIST. At most, use TextFormat::RESET
```

### 3.9 Items and blocks by numeric ID

On MCPE 0.15.10 everything is built with numeric IDs from the era. String IDs
(`"minecraft:stone"`, modern `ItemTypeIds::...`) do not exist yet.

```php
use pocketmine\item\Item;
use pocketmine\block\Block;

$sword = Item::get(276, 0, 1);   // diamond sword
$stone = Item::get(1, 0, 64);    // 64 stone
$block = Block::get(49, 0);      // obsidian
```

Era signatures: `Item::get($id, $meta = 0, $count = 1)` and `Block::get($id, $meta = 0)`.
The second parameter is meta / damage (for example wool color, 0-15).

Common IDs of the 0.15 era:

| ID | Object |
| -- | ------ |
| 1 | Stone |
| 4 | Cobblestone |
| 17 | Log |
| 20 | Glass |
| 46 | TNT |
| 49 | Obsidian |
| 54 | Chest |
| 57 | Diamond block |
| 261 | Bow |
| 262 | Arrow |
| 264 | Diamond |
| 265 | Iron ingot |
| 276 | Diamond sword |
| 278 | Diamond pickaxe |
| 280 | Stick |
| 297 | Bread |
| 322 | Golden apple |

If you need an ID that is not in the table, declare it as a commented constant inside
your plugin instead of "guessing" a modern string.

### 3.10 Common Player and Level API

| What you need | Era call |
| ------------- | -------- |
| Name | `$player->getName()` |
| Display name | `$player->getDisplayName()` |
| Private message | `$player->sendMessage($text)` |
| Permissions | `$player->hasPermission("mikitoplugin.usar")` |
| Game mode | `$player->getGamemode()` returns `int` (0 survival, 1 creative, 2 adventure) |
| Change mode | `$player->setGamemode(1)` |
| Current world | `$player->getLevel()` |
| Teleport | `$player->teleport(Vector3 $position)` |
| Inventory | `$player->getInventory()->setItemInHand(Item::get(276))` |
| Disconnect | `$player->kick($reason)` |
| Server from the plugin | `$this->getServer()` (no need for `Server::getInstance()`) |
| Plugin data folder | `$this->getDataFolder()` |
| Logger | `$this->getLogger()->info($text)` |
| World by name | `$this->getServer()->getLevelByName("world")` |
| Block at position | `$level->getBlock(new Vector3($x, $y, $z))` |

Notes:

1. Era method names use the spelling `getGamemode()` (lowercase m); do not "fix" it to
   `getGameMode()` if it does not exist.
2. Positions are built with `pocketmine\math\Vector3` or with
   `pocketmine\level\Location` when the world must be specified.
3. When you get the player from an event, prefer `$event->getPlayer()` over name
   lookups against the server.

### 3.11 Note about the era's forks

Genisys (iTXTech) and ImagicalMine were contemporary forks of PocketMine-MP 2.0.0
with an almost identical API. It is valid for a plugin built under this constitution
to run on them, but do not code against fork-exclusive APIs: keep the lowest common
denominator (the namespaces and signatures documented here) so the same code runs on
PocketMine 2.0.0, Genisys and ImagicalMine unchanged.
Ideally Genisys is preferred, since it is the fork that uses PHP 7.0.x.

## 4. Architecture and quality

### 4.1 Canonical plugin structure

Split by responsibility; a single giant file is the most expensive anti-pattern to
maintain:

```text
MiPlugin/
├── plugin.yml
└── src/
    └── mikitoplugin/
        ├── Main.php
        ├── command/
        │   └── PuntosCommand.php
        ├── listener/
        │   └── JugadorListener.php
        └── task/
            └── ResumenTask.php
```

Rules:

1. `Main extends PluginBase` only orchestrates: it registers listeners, commands and
   tasks.
2. One command per file in `command/`, one listener per file in `listener/`, one task
   per file in `task/`.
3. Avoid god classes: if `Main.php` grows past a few dozen lines of logic (not
   registration), extract the logic into its own classes.
4. The namespace mirrors the folders: `mikitoplugin\command\PuntosCommand` lives in
   `src/mikitoplugin/command/PuntosCommand.php` (PSR-4).

### 4.2 onEnable as the single entry point

1. Everything that "turns the plugin on" (events, commands, tasks, configs) happens in
   `onEnable()`.
2. The constructor registers nothing and never queries the server.
3. `onLoad()` stays free of heavy logic; if you do not need it, do not declare it.
4. `onDisable()` cancels tasks and persists state.
5. Initialization must be idempotent: an enable after a disable must not duplicate
   registrations.

### 4.3 2016-era anti-patterns

This is what LLMs tend to generate (and what broke 2016 servers):

1. `eval()`, dynamic `assert()` or any runtime code execution: forbidden.
2. SQL queries concatenating unescaped variables: always prepared statements
   (`SQLite3::prepare()` was the typical path) or at least `escapeString()`.
3. Player state keyed by raw `$player->getName()`: names change capitalization. Use
   `strtolower($player->getName())` as the canonical key.
4. Storing references to `Player` objects in long-lived collections: when the player
   disconnects, ghosts remain in memory. Store names (lowercase), not objects.
5. `sleep()` or long loops inside listeners or `onRun()`: they freeze the tick loop and
   everyone disconnects.
6. A god class of thousands of lines mixing commands, events, tasks and SQL.
7. Copy-pasting modern StackOverflow code without checking the PHP version: the number
   one source of 7.1+ leaks.
8. `chmod 0777` on plugin files or the data folder.
9. Global state in static variables shared across plugin instances.
10. Breaking the `plugin.yml` contract (folder name mismatch, `main` pointing to a
    missing class, no declared `api`).

### 4.4 Code conventions

1. PSR-1: one file = one class = one namespace; `<?php` with no closing `?>`; no side
   effects when including files.
2. PSR-2: 4-space indentation (never tabs); opening brace on its own line for classes
   and methods and on the same line for control structures; blank line after the
   `namespace` declaration; visibility declared on every method and property.
3. Names: `PascalCase` for classes, `camelCase` for methods, properties and variables,
   `UPPER_SNAKE_CASE` for constants, lowercase namespaces.
4. PSR-4: the folder structure mirrors the namespace (see 4.1).
5. Era-style docblocks: `@param`, `@return` and `@var` carrying the types PHP 7.0
   cannot express (`null`, resources, callbacks).
6. User-facing strings in the server's language, colored through `TextFormat` and not
   with raw `\x00` bytes.

### 4.5 Security rules

1. Never trust `$args`: validate count (`count($args)`), format (`is_numeric`,
   `ctype_digit`) and membership in an allowed set (`in_array` against a whitelist)
   before using any value.
2. Values read from `Config` arrive with no type guarantees: sanitize them and apply
   min/max ranges before using them in loops, timers or numeric limits.
3. Every command declares a permission in `plugin.yml` and checks it with
   `testPermission()` or `hasPermission()` before running logic.
4. If there is SQL, go through prepared statements; building the query string by
   concatenating input is forbidden.
5. If there are file paths, build them inside `getDataFolder()` applying `basename()`
   to any user input: no `../` and no absolute paths.
6. Never show players internal error messages, stack traces or system paths: log them
   in the plugin logger and return a generic message.
7. Strip color codes from user input before relaying it (chat, signs, books) with
   `TextFormat::RESET` to prevent format injection.
8. If credentials must be hashed, use `password_hash()` / `password_verify()` (both
   available in 7.0); never raw MD5 or SHA1.
9. Never store secrets or tokens in `plugin.yml` or in the source code.

## 5. Agent behavior rules

1. **This document > your memory.** For any API doubt, prefer the PM2 style documented
   here over your "recollection" of PM3/4/5.
2. **Exactly PHP 7.0.14.** If when reviewing your own output you spot 7.1+ syntax
   (`void`, `?type`, `fn`, `match`, `?->`, `??=`...), rewrite it before delivering.
   Never deliver code "tested on PHP 8".
3. **Zero Composer.** Do not generate `composer.json`, `composer.lock` or Packagist
   dependencies: the era's workflow is `plugin.yml` + phar/DevTools. If a library is
   essential, vendor its source inside the plugin with its license.
4. **Always plugin.yml.** Every PHP code delivery ships with its complete `plugin.yml`
   (`name`, `version`, `main`, `api: [2.0.0]`, plus `commands` and `permissions` when
   applicable).
5. **Do not invent APIs.** If a detail is not in this document and you are not certain,
   write your best recollection and mark it with `[VERIFICAR]` in a comment so the
   maintainer can double-check it; that must not block delivery.
6. **Compilable snippets.** Every example labeled as correct must compile on
   PHP 7.0.14: review it mentally line by line before delivering.
7. **No modern shortcuts.** No string item IDs, no `Vanilla*` classes, no PHP 8 helpers
   that "replace" era code with fewer lines.
8. **The stack is sacred.** If the user requests an API that does not exist on 0.15.10
   (mobs, blocks or commands from later versions), explain the limitation and offer the
   era-compatible alternative instead of forcing the modern API.
9. **Mandatory closing step.** Before ending your response, walk through the
   SELF-CHECK in section 6 and fix everything that fails.
10. **Language sync.** If you edit this file, apply the same change to
    [../es/AGENTS.md](../es/AGENTS.md).

## 6. SELF-CHECK (mandatory checklist before delivering)

Tick every item. If one fails, do not deliver: fix it and run the list again.

- [ ] No `: void`, no `?type`, no `iterable`, no `catch (A | B $e)` in the code (7.1+).
- [ ] No `[$a, $b] =`, no `list()` with keys, no `public const`, no `text[-1]` (7.1+).
- [ ] No typed property, no `fn(`, no `??=`, no `[...` spread, no `object` (7.2-7.4).
- [ ] No `match (`, no named argument, no promoted constructor properties, no `#[`, no `enum`, no `readonly`, no `mixed`, no `never`, no union types (8.x).
- [ ] No `str_contains(`, `str_starts_with(`, `str_ends_with(`, `get_debug_type(`, `?->`, no `throw` as an expression (8.x).
- [ ] Zero modern PocketMine namespaces (`pocketmine\player\`, `pocketmine\world\`) and no `World`, `VanillaItems`, `VanillaBlocks`, `ItemTypeIds` classes.
- [ ] All world access uses `Level` and `$player->getLevel()`.
- [ ] Items and blocks use 0.15-era numeric IDs; zero `"minecraft:..."` strings.
- [ ] `Task::onRun($currentTick)` has no return type.
- [ ] No logic in constructors: the whole boot sequence lives in `onEnable()`.
- [ ] No `composer.json` and no Packagist dependencies were generated.
- [ ] The delivered `plugin.yml` is complete and declares `api: [2.0.0]`.
- [ ] Player state is keyed by `strtolower($player->getName())` and stores no `Player` objects.
- [ ] Zero `eval()`, zero concatenated SQL, zero `sleep()` inside the tick loop.
- [ ] Every command validates `$args` and checks permissions before acting.
- [ ] The code follows PSR-2 (4 spaces, no tabs, convention-correct braces).
- [ ] Every API doubt is marked with `[VERIFICAR]` in a comment.
- [ ] If you touched this file, you also updated [es/AGENTS.md](../es/AGENTS.md).
