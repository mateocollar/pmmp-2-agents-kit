# AGENTS.md — PocketMine-MP 2.0.0 (constitución para agentes de IA)

> Versión maestra en español. La traducción oficial vive en [en/AGENTS.md](../en/AGENTS.md).
> Todo cambio de contenido aplica a ambos archivos: si tocás uno, tocá el otro.

## 1. Contexto del stack

Este documento es la constitución que gobierna todo código generado para este proyecto.
El stack objetivo es fijo, sagrado e innegociable:

| Componente | Versión obligatoria |
| ---------- | ------------------- |
| PocketMine-MP | 2.0.0 (API `2.0.0`) |
| Minecraft Pocket Edition | 0.15.10 (protocolo 84) |
| PHP | 7.0.14 exactamente |
| Época de referencia | 2016 |
| Formato de plugin | `plugin.yml` + phar / DevTools de la era |

### 1.1 Advertencia obligatoria al agente

Tu conocimiento por defecto está sesgado a PocketMine moderno (PM3, PM4, PM5) y a
PHP 7.2-8.x, porque son las versiones con más ejemplos entrenados. **Todo lo que sigue
prevalece sobre tu conocimiento entrenado.** Si un recuerdo tuyo de una API contradice
este documento, gana este documento. Si dudás entre "lo que recuerdo" y "lo que dice la
constitución", escribí lo que dice la constitución.

Señales típicas de que estás equivocado: usaste `pocketmine\player\Player`, `World`,
`getWorld()`, IDs string de items, `: void`, `?->`, `match`, `fn()`, `str_contains()`
o cualquier cosa que compile en PHP 8 pero no en 7.0.14. Revisá y reescribí antes de
entregar.

### 1.2 Cómo usar este documento

1. Leé la sección 2 antes de escribir una sola línea de PHP: las restricciones de
   PHP 7.0 son lo que más rompe en este stack.
2. Leé la sección 3 antes de usar cualquier clase de PocketMine: los namespaces y
   firmas de la era 2.0.0 no son los que aparecen en tutoriales modernos.
3. Seguí las reglas de la sección 4 (arquitectura) y de la sección 5 (comportamiento
   del agente).
4. Antes de entregar, recorré ítem por ítem la checklist de la sección 6 (SELF-CHECK).
   Si un ítem falla, no entregues: corregí.

## 2. Restricciones de PHP 7.0.14

### 2.1 Regla general

El servidor corre sobre PHP 7.0.14 con pthreads de la época. Cualquier sintaxis
introducida en 7.1 o posterior produce un error de parseo fatal al cargar el plugin y
el servidor directamente no lo levanta. No existe "funciona igual": o compila en 7.0.14
o no existe.

### 2.2 Tabla PROHIBIDO y PERMITIDO

| Sintaxis o recurso | Desde | Estado | Alternativa válida en PHP 7.0 |
| ------------------ | ----- | ------ | ----------------------------- |
| `void` (tipo de retorno o hint) | 7.1 | PROHIBIDO | omitir el tipo y documentar con `@return void` |
| `?int`, `?string` (tipos anulables) | 7.1 | PROHIBIDO | sin tipo y docblock `@param tipo\|null` |
| `iterable` | 7.1 | PROHIBIDO | `array` o sin tipo |
| `catch (A \| B $e)` (multi-catch) | 7.1 | PROHIBIDO | dos bloques `catch` encadenados |
| `[$a, $b] = $arr` (destructuring corto) | 7.1 | PROHIBIDO | `list($a, $b) = $arr` |
| `list("clave" => $v) = $arr` | 7.1 | PROHIBIDO | asignación manual `$v = $arr["clave"];` |
| `public const X` (visibilidad en constantes) | 7.1 | PROHIBIDO | `const X` |
| `$texto[-1]` (offset negativo) | 7.1 | PROHIBIDO | `substr($texto, -1)` |
| `object` (tipo) | 7.2 | PROHIBIDO | sin tipo y docblock |
| `array_key_first()` / `array_key_last()` | 7.3 | PROHIBIDO | `reset($arr)` y `end($arr)` con `key()` |
| heredoc flexible (marcador indentado) | 7.3 | PROHIBIDO | marcador de cierre al inicio de la línea |
| coma final en llamadas `f($a, $b,)` | 7.3 | PROHIBIDO | quitar la coma (en arrays sí se permite) |
| propiedades tipadas `private int $x` | 7.4 | PROHIBIDO | `private $x;` con docblock `@var` |
| arrow functions `fn($x) => ...` | 7.4 | PROHIBIDO | `function ($x) { ... }` |
| asignación coalescente `??=` | 7.4 | PROHIBIDO | `if (!isset($x)) { $x = ...; }` |
| spread en arrays `[...$a]` | 7.4 | PROHIBIDO | `array_merge($a, $b)` |
| `__serialize()` / `__unserialize()` | 7.4 | PROHIBIDO | propiedades normales con `serialize()` |
| `match` | 8.0 | PROHIBIDO | `switch` |
| argumentos con nombre `f(cond: $x)` | 8.0 | PROHIBIDO | argumentos posicionales |
| promoción de propiedades en constructores | 8.0 | PROHIBIDO | declarar la propiedad y asignarla en el cuerpo |
| atributos `#[Algo]` | 8.0 | PROHIBIDO | docblocks (`@param`, `@priority`) |
| union types `int\|string` | 8.0 | PROHIBIDO | sin tipo y validar con `is_int()` / `is_string()` |
| `mixed` / `never` | 8.0 / 8.1 | PROHIBIDO | sin tipo y docblock |
| `str_contains()` / `str_starts_with()` / `str_ends_with()` | 8.0 | PROHIBIDO | `strpos($s, $b) !== false` |
| operador `?->` | 8.0 | PROHIBIDO | comprobación explícita `if ($x !== null)` |
| `throw` como expresión | 8.0 | PROHIBIDO | `throw` dentro de una sentencia propia |
| `get_debug_type()` / `array_is_list()` | 8.0 / 8.1 | PROHIBIDO | `is_*()` y `gettype()` |
| `enum` | 8.1 | PROHIBIDO | clase con `const` |
| `readonly` | 8.1 | PROHIBIDO | propiedad privada + getter |
| scalar type hints `int`, `string`, `bool`, `float` | 7.0 | PERMITIDO | — |
| tipos de retorno básicos `int`, `string`, `bool`, `float`, `array`, nombre de clase | 7.0 | PERMITIDO | — |
| operador nulo coalescente `??` | 7.0 | PERMITIDO | — |
| operador nave espacial `<=>` | 7.0 | PERMITIDO | — |
| anonymous classes | 7.0 | PERMITIDO | — |
| variadics `...$args` | 7.0 | PERMITIDO | — |
| closures clásicas `function () use ($x) { ... }` | 7.0 | PERMITIDO | — |
| `yield from`, `**`, `random_bytes()`, `intdiv()` | 7.0 | PERMITIDO | — |

### 2.3 Qué se permite usar en PHP 7.0.14

PHP 7.0 ya trae suficiente para escribir plugins cómodos. Ejemplo incorrecto, no compilable
tal cual en 7.0.14:

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

### 2.4 Síntomas de código moderno (snippets)

Si tu salida contiene alguna de estas pistas, estás escribiendo el PHP equivocado.
Reescribí antes de entregar. Los fragmentos marcados como PROHIBIDO son ilustrativos y
a propósito NO compilan en 7.0.14; los fragmentos marcados como CORRECTOS sí compilan.

Par 1 — retorno `void`, el error más frecuente en tareas de PocketMine:

```php
// PROHIBIDO — PHP 7.1+
public function onRun($currentTick): void
{
}

// CORRECTO — PHP 7.0.14
public function onRun($currentTick)
{
}
```

Par 2 — tipos anulables:

```php
// PROHIBIDO — PHP 7.1+
public function buscar(?string $nombre): ?string
{
    return $nombre;
}

// CORRECTO — PHP 7.0.14
/**
 * @param string|null $nombre
 * @return string|null
 */
public function buscar($nombre)
{
    return $nombre;
}
```

Par 3 — operador nulo seguro:

```php
// PROHIBIDO — PHP 8.0
$nick = $jugador?->getDisplayName();

// CORRECTO — PHP 7.0.14
$nick = "Jugador";
if ($jugador !== null) {
    $nick = $jugador->getDisplayName();
}
```

Par 4 — `match` contra `switch`:

```php
// PROHIBIDO — PHP 8.0
$etiqueta = match ($modo) {
    0 => "Supervivencia",
    1 => "Creativo",
    default => "Otro",
};

// CORRECTO — PHP 7.0.14
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

Par 5 — arrow function contra closure:

```php
// PROHIBIDO — PHP 7.4
$online = array_filter($todos, fn($p) => $p->isOnline());

// CORRECTO — PHP 7.0.14
$online = array_filter($todos, function ($p) {
    return $p->isOnline();
});
```

Par 6 — helpers de string de PHP 8:

```php
// PROHIBIDO — PHP 8.0
if (str_contains($nombre, "admin")) {
    $esAdmin = true;
}

// CORRECTO — PHP 7.0.14
if (strpos($nombre, "admin") !== false) {
    $esAdmin = true;
}
```

Par 7 — constructor promotion contra propiedades explícitas:

```php
// PROHIBIDO — PHP 8.0
class Entrada
{
    public function __construct(
        public string $jugador,
        public int $puntos = 0
    ) {
    }
}

// CORRECTO — PHP 7.0.14
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

Par 8 — destructuring y multi-catch:

```php
// PROHIBIDO — PHP 7.1+
[$x, $y] = $coordenadas;
try {
    guardar($datos);
} catch (RuntimeException | InvalidArgumentException $e) {
    registrar($e->getMessage());
}

// CORRECTO — PHP 7.0.14
list($x, $y) = $coordenadas;
try {
    guardar($datos);
} catch (RuntimeException $e) {
    registrar($e->getMessage());
} catch (InvalidArgumentException $e) {
    registrar($e->getMessage());
}
```

Par 9 — propiedades tipadas, `??=` y spread en arrays:

```php
// PROHIBIDO — PHP 7.4+
class Contador
{
    private int $valor = 0;
}
$config["idioma"] ??= "es";
$todo = [...$base, ...$extra];

// CORRECTO — PHP 7.0.14
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

Par 10 — enum contra clase de constantes:

```php
// PROHIBIDO — PHP 8.1
enum Prioridad
{
    case Baja;
    case Alta;
}

// CORRECTO — PHP 7.0.14
class Prioridad
{
    const BAJA = 0;
    const ALTA = 1;
}
```

Par 11 — constante con visibilidad y offset negativo:

```php
// PROHIBIDO — PHP 7.1+
public const MAXIMO = 10;
$ultimo = $nombre[-1];

// CORRECTO — PHP 7.0.14
const MAXIMO = 10;
$ultimo = substr($nombre, -1);
```

### 2.5 declare(strict_types=1)

`declare(strict_types=1)` es **a criterio del plugin**: no es obligatorio y el stack no
lo exige. Si lo usás, respetá estas dos reglas:

1. Debe ser la primera sentencia del archivo, antes de `namespace` y de cualquier `use`.
2. Aplique a todo el archivo de forma consistente: no mezcles archivos con y sin
   strict_types dentro del mismo plugin.

Advertencia práctica de la era: muchas APIs de PocketMine 2.0.0 devuelven valores
"flojos" (por ejemplo enteros que llegan desde YAML o desde la red). Con
strict_types, una comparación o asignación estricta puede lanzar un `TypeError` en
producción. Si no estás seguro, dejalo fuera y validá tipos a mano con `is_int()`,
`is_numeric()` y casting explícito `(int)` / `(string)`.

## 3. Restricciones de PocketMine-MP 2.0.0

### 3.1 Namespaces planos (la regla de oro)

En la API 2.0.0 los nombres de clase viven en namespaces planos y en minúsculas de
carpeta. **No existen** `pocketmine\player\Player`, `pocketmine\world\World` ni
`pocketmine\server\Server`: esas son renombrados de versiones modernas.

| Ámbito | Namespace correcto en PM 2.0.0 |
| ------ | ------------------------------ |
| Jugador y servidor | `pocketmine\Player`, `pocketmine\Server` |
| Mundos (levels) | `pocketmine\level\Level`, `pocketmine\level\Position`, `pocketmine\level\Location` |
| Items y bloques | `pocketmine\item\Item`, `pocketmine\block\Block` |
| NBT | `pocketmine\nbt\tag\CompoundTag`, `pocketmine\nbt\tag\StringTag`, `pocketmine\nbt\tag\IntTag` |
| Plugins | `pocketmine\plugin\PluginBase`, `pocketmine\plugin\PluginManager` |
| Comandos | `pocketmine\command\Command`, `pocketmine\command\CommandSender` |
| Eventos | `pocketmine\event\Listener`, `pocketmine\event\player\*`, `pocketmine\event\block\*` |
| Scheduler | `pocketmine\scheduler\Task`, `pocketmine\scheduler\TaskHandler` |
| Utilidades | `pocketmine\utils\Config`, `pocketmine\utils\TextFormat` |
| Matemáticas | `pocketmine\math\Vector3` |

Reglas derivadas:

1. El mundo se llama `Level`, nunca `World`. `$player->getLevel()`, nunca
   `$player->getWorld()`.
2. `TextFormat` vive en `pocketmine\utils\TextFormat` y sus constantes son
   `TextFormat::GREEN`, `TextFormat::RED`, `TextFormat::BOLD`... sin prefijos tipo
   `FORMAT_` y sin métodos estáticos "de utilidad moderna".
3. Los items y bloques se referencian por ID numérico de la era 0.15. Los IDs string
   como `"minecraft:stone"` todavía no existen.

### 3.2 Tabla de mapeo PM5 a PM2

Si "recordás" una API de PocketMine moderno, usá la columna derecha:

| Si tu memoria dice (PM3/PM4/PM5) | Escribí en PM 2.0.0 |
| -------------------------------- | ------------------- |
| `pocketmine\player\Player` | `pocketmine\Player` |
| `pocketmine\server\Server` | `pocketmine\Server` |
| `pocketmine\world\World` | `pocketmine\level\Level` |
| `pocketmine\world\Position` | `pocketmine\level\Position` |
| `pocketmine\world\Location` | `pocketmine\level\Location` |
| `$player->getWorld()` | `$player->getLevel()` |
| `$world->getBlock(...)` | `$level->getBlock(...)` |
| `VanillaItems::DIAMOND_SWORD()` | `Item::get(276)` |
| `VanillaBlocks::STONE()` | `Block::get(1)` |
| `ItemTypeIds::STONE` / `"minecraft:stone"` | `1` (ID numérico) |
| `$player->getGamemode()` devuelve enum `GameMode` | `$player->getGamemode()` devuelve `int` (0, 1, 2) |
| `GameMode::SURVIVAL()` | `0` (`Player::SURVIVAL` [VERIFICAR]) |
| `Task::onRun()` sin argumentos | `Task::onRun($currentTick)` |
| `plugin.yml` con `api: [5.0.0]` | `plugin.yml` con `api: [2.0.0]` |
| `pocketmine\event\player\PlayerJoinEvent` | igual, sin cambio |
| `pocketmine\utils\TextFormat` | igual, sin cambio |

### 3.3 plugin.yml de la época

Todo plugin entrega su `plugin.yml` completo en la raíz del plugin. Ejemplo mínimo
correcto de la era:

```yaml
name: MiPlugin
version: 1.0.0
main: mikitoplugin\Main
api: [2.0.0]
author: TuNombre
description: Plugin de ejemplo para PocketMine-MP 2.0.0
commands:
  puntos:
    description: Muestra tus puntos
    usage: "/puntos"
    permission: mikitoplugin.puntos
permissions:
  mikitoplugin.puntos:
    description: Permite usar el comando /puntos
    default: true
```

Notas de la época:

1. `api` se declara como lista de la era (`api: [2.0.0]`); no mezcles versiones de API
   modernas.
2. `main` es el nombre completo de la clase con separadores `\`, sin comillas dobles
   (en YAML comillas dobles convertirían `\` en escape).
3. El valor de `main` apunta a la clase cuyo archivo vive bajo `src/` con estructura
   PSR-4 cuando se usa DevTools en modo fuente; en un phar compilado las clases quedan
   en la raíz. Verificá la variante que use tu proyecto.
4. No agregues campos de plugin.yml modernos (`loadbefore`, `creators`...) que la
   parser de la época desconoce.

### 3.4 Ciclo de vida y estructura de un plugin

La clase principal extiende `PluginBase` y suele implementar `Listener` a la vez.
Ejemplo completo y compilable en PHP 7.0.14:

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
        // PluginBase no expone getScheduler()
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
            TextFormat::GREEN . "Bienvenido, " . TextFormat::WHITE . $event->getPlayer()->getName()
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

Reglas de ciclo de vida:

1. `onEnable()` es el único punto de arranque: ahí se registran eventos, comandos y
   tareas, y se carga la configuración.
2. El constructor de `PluginBase` no se sobreescribe con lógica: solo inicialización
   trivial de propiedades si hace falta. Nunca toques el servidor desde un constructor.
3. `onDisable()` libera estado y cancela tareas pendientes.
4. Los métodos de ciclo de vida se declaran sin tipo de retorno (PHP 7.0 no tiene
   `void`).

### 3.5 Comandos

Los comandos de la era son clases que extienden `pocketmine\command\Command` e
implementan `execute()`. Ejemplo compilable:

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
        parent::__construct("puntos", "Muestra tus puntos", "/puntos", ["pt"]);
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
            $sender->sendMessage(TextFormat::RED . "Uso: /puntos [jugador]");
            return;
        }
        $nombre = count($args) === 1 ? $args[0] : $sender->getName();
        $total = $this->plugin->sumarPuntos($nombre, 0);
        $sender->sendMessage(TextFormat::AQUA . $nombre . TextFormat::WHITE . " tiene " . $total . " puntos");
    }
}
```

Registro en `Main::onEnable()`:

```php
$this->getServer()->getCommandMap()->register("mikitoplugin", new PuntosCommand($this));
```

Notas:

1. `setPermission()` y `testPermission()` siguen el estilo Bukkit de la época.
2. También podés declarar el comando en `plugin.yml` bajo `commands:`; Usando `CommandExecute` y el manejo se delega a tu clase de
   comando.
3. `execute()` no lleva tipo de retorno y su cuerpo valida siempre `$args` y permisos
   antes de hacer nada.

### 3.6 Eventos

Los listeners implementan `pocketmine\event\Listener`. Cada handler es un método
público con un único parámetro tipeado con el evento y la metadata en docblock:

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

Reglas:

1. El registro ocurre una sola vez en `onEnable()` con
   `$this->getServer()->getPluginManager()->registerEvents($this, $this);`.
2. `@priority` acepta `LOWEST`, `LOW`, `NORMAL`, `HIGH`, `HIGHEST` y `MONITOR`.
3. `@ignoreCancelled` acepta `true` o `false`; si el evento puede cancelarse y te
   importa el resultado, usá `true`.
4. Handlers sin docblock de prioridad corren con la prioridad por defecto de la era;
   siempre declara la intención de forma explícita.
5. No hagas trabajo pesado dentro de handlers de alto volumen (`PlayerMoveEvent`
   dispara decenas de veces por segundo): delegá a una tarea del scheduler.

Eventos más usados de la era:

| Evento | Namespace | Métodos útiles |
| ------ | --------- | -------------- |
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

Los métodos `setCancelled()` / `isCancelled()` existen en los eventos cancelables de la
era; los cancelados no deben mutar estado del servidor.

### 3.7 Scheduler y tareas

Las tareas periódicas son clases que extienden `pocketmine\scheduler\Task` y
sobrescriben `onRun()` **sin tipo de retorno** (recuerda: `void` no existe en PHP 7.0):

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
        $this->plugin->getLogger()->info("Resumen ejecutado en tick " . $currentTick);
    }
}
```

API del scheduler de la era:

| Operación | Llamada |
| --------- | ------- |
| Repetir cada N ticks | `$scheduler->scheduleRepeatingTask($task, 20)` |
| Ejecutar tras N ticks | `$scheduler->scheduleDelayedTask($task, 100)` |
| Cancelar | `$handler->cancel();` en el `TaskHandler` devuelto [VERIFICAR] |

Notas:

1. 20 ticks equivalen a un segundo; todo lo que "parezca tiempo" se mide en ticks.
2. `onRun($currentTick)` recibe el tick actual; no le agregues tipo de retorno ni
   parámetros extra.
3. Nunca uses `sleep()`, bucles largos ni `file_get_contents()` bloqueante dentro de
   `onRun()` o de un listener: congelás el tick loop de todo el servidor.
4. `AsyncTask` y la locura de pthreads existen en la era, pero para plugins simples
   complican más de lo que ayudan: regla v1, tareas síncronas nomás.
5. Guardá el `TaskHandler` si necesitás cancelar la tarea en `onDisable()`.

### 3.8 Config y TextFormat

`pocketmine\utils\Config` es el almacén estándar de configuración y estado pequeño:

```php
<?php

namespace mikitoplugin;

use pocketmine\utils\Config;

// dentro de onEnable()
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

Notas:

1. Los tipos de `Config` de la época son `Config::YAML` como uso
   principal, además de variantes serializadas de la propia clase.
2. `Config::get($clave, $defecto)` devuelve lo que guardaste sin garantías de tipo:
   siempre validá con `is_int()`, `is_numeric()` o `is_string()` antes de usarlo en
   cálculos.
3. El archivo vive en `$this->getDataFolder()`; nunca escribas fuera de esa carpeta.

Colores y formato con `pocketmine\utils\TextFormat`:

```php
use pocketmine\utils\TextFormat;

$player->sendMessage(TextFormat::GREEN . "Puntos: " . TextFormat::WHITE . $puntos);
$limpio = TextFormat::clean($mensajeDelUsuario); // Este metodo NO EXISTE. Como mucho, usa TextFormat::RESET
```

### 3.9 Items y bloques por ID numérico

En MCPE 0.15.10 todo se construye con IDs numéricos de la época. Los IDs string
(`"minecraft:stone"`, `ItemIds::...` de versiones modernas) no existen todavía.

```php
use pocketmine\item\Item;
use pocketmine\block\Block;

$espada = Item::get(276, 0, 1);   // espada de diamante
$piedra = Item::get(1, 0, 64);    // 64 de piedra
$bloque = Block::get(49, 0);      // obsidiana
```

Firma de la era: `Item::get($id, $meta = 0, $count = 1)` y `Block::get($id, $meta = 0)`. El segundo parámetro es el meta / daño (por ejemplo el
color de la lana, 0-15).

IDs de uso común en la era 0.15:

| ID | Objeto |
| -- | ------ |
| 1 | Piedra |
| 4 | Adoquín |
| 17 | Tronco |
| 20 | Vidrio |
| 46 | TNT |
| 49 | Obsidiana |
| 54 | Cofre |
| 57 | Bloque de diamante |
| 261 | Arco |
| 262 | Flecha |
| 264 | Diamante |
| 265 | Lingote de hierro |
| 276 | Espada de diamante |
| 278 | Pico de diamante |
| 280 | Palo |
| 297 | Pan |
| 322 | Manzana dorada |

Si necesitás un ID que no está en la tabla, declaralo como constante con comentario en
tu plugin en lugar de "adivinar" un string moderno.

### 3.10 API común de Player y Level

| Qué necesitás | Llamada de la era |
| ------------- | ----------------- |
| Nombre | `$player->getName()` |
| Nombre mostrado | `$player->getDisplayName()` |
| Mensaje privado | `$player->sendMessage($texto)` |
| Permisos | `$player->hasPermission("mikitoplugin.usar")` |
| Modo de juego | `$player->getGamemode()` devuelve `int` (0 supervivencia, 1 creativo, 2 aventura) |
| Cambiar modo | `$player->setGamemode(1)` |
| Mundo actual | `$player->getLevel()` |
| Teletransportar | `$player->teleport(Vector3 $position)` |
| Inventario | `$player->getInventory()->setItemInHand(Item::get(276))` |
| Desconectar | `$player->kick($motivo)` |
| Servidor desde el plugin | `$this->getServer()` (no hace falta `Server::getInstance()`) |
| Carpeta del plugin | `$this->getDataFolder()` |
| Logger | `$this->getLogger()->info($texto)` |
| Mundo por nombre | `$this->getServer()->getLevelByName("world")` |
| Bloque en posición | `$level->getBlock(new Vector3($x, $y, $z))` |

Notas:

1. Los nombres de métodos de la época usan la grafía `getGamemode()` (m minúscula);
   no la "corrijas" a `getGameMode()` si no existe.
2. Las posiciones se construyen con `pocketmine\math\Vector3` o con
   `pocketmine\level\Location` cuando hace falta especificar mundo.
3. Al recibir el jugador de un evento, preferí `$event->getPlayer()` antes que
   búsquedas por nombre contra el servidor.

### 3.11 Nota sobre los forks de la época

Genisys (iTXTech) y ImagicalMine fueron forks contemporáneos de PocketMine-MP 2.0.0
con una API prácticamente idéntica. Es válido que un plugin de esta constitución
funcione en ellos, pero no codees contra APIs exclusivas de un fork: mantené el mínimo
común denominador (los namespaces y firmas documentados acá) para que el mismo código
corra en PocketMine 2.0.0, Genisys e ImagicalMine sin cambios.
Idealmente se prefiere Genisys, ya que es el fork que usa PHP 7.0.x

## 4. Arquitectura y calidad

### 4.1 Estructura canónica del plugin

Separá por responsabilidades; un solo archivo gigante es el anti-patrón más caro de
mantener:

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

Reglas:

1. `Main extends PluginBase` solo orquesta: registra listeners, comandos y tareas.
2. Un comando por archivo en `command/`, un listener por archivo en `listener/`, una
   tarea por archivo en `task/`.
3. Evitá clases-dios: si `Main.php` pasa de unas pocas decenas de líneas de lógica
   (no de registro), extráe la lógica a clases propias.
4. El namespace espeja las carpetas: `mikitoplugin\command\PuntosCommand` vive en
   `src/mikitoplugin/command/PuntosCommand.php` (PSR-4).

### 4.2 onEnable como único punto de arranque

1. Todo lo que "prende" el plugin (eventos, comandos, tareas, configs) ocurre en
   `onEnable()`.
2. El constructor no registra nada y no consulta el servidor.
3. `onLoad()` queda libre de lógica pesada; si no lo necesitás, no lo declares.
4. `onDisable()` cancela tareas y guarda estado persistente.
5. La inicialización debe ser idempotente: un enable tras un disable no debe duplicar
   registros.

### 4.3 Anti-patrones de la era 2016

Esto es lo que los LLM tienden a generar (y lo que rompía los servidores de 2016):

1. `eval()`, `assert()` dinámico o cualquier ejecución de código en runtime: prohibido.
2. Queries SQL concatenando variables sin escapar: siempre prepared statements
   (`SQLite3::prepare()` era el camino típico) o, como mínimo, `escapeString()`.
3. Guardar estado de jugadores usando `$player->getName()` crudo como clave: los
   nombres cambian de mayúsculas. Usá `strtolower($player->getName())` como clave
   canónica.
4. Guardar referencias a objetos `Player` en colecciones de larga vida: cuando el
   jugador se desconecta quedan fantasmas en memoria. Guardá nombres (en minúsculas),
   no objetos.
5. `sleep()` o bucles largos en listeners o en `onRun()`: congelan el tick loop y
   desconectan a todos.
6. Clase-dios de miles de líneas con comandos, eventos, tareas y SQL mezclados.
7. Copiar y pegar código de StackOverflow de versiones modernas sin revisar la versión
   de PHP: la fuente número uno de fugas 7.1+.
8. `chmod 0777` sobre archivos del plugin o de la data folder.
9. Estado global en variables estáticas compartidas entre instancias del plugin.
10. Romper el contrato de `plugin.yml` (nombre distinto de la carpeta, `main` apuntando
    a una clase inexistente, `api` sin declarar).

### 4.4 Convenciones de código

1. PSR-1: un archivo = una clase = un namespace; `<?php` sin `?>` de cierre; sin
   efectos secundarios al incluir archivos.
2. PSR-2: indentación de 4 espacios (nunca tabs); llave de apertura en línea nueva
   para clases y métodos y en la misma línea para estructuras de control; línea en
   blanco después de la declaración de `namespace`; visibilidad declarada en cada
   método y propiedad.
3. Nombres: `PascalCase` para clases, `camelCase` para métodos, propiedades y
   variables, `UPPER_SNAKE_CASE` para constantes, namespaces en minúsculas.
4. PSR-4: la estructura de carpetas espeja el namespace (ver 4.1).
5. Docblocks al estilo 2016: `@param`, `@return` y `@var` con los tipos que PHP 7.0
   no puede expresar (`null`, recursos, callbacks).
6. Strings de usuario visibles en español si el servidor lo está, con códigos de color
   vía `TextFormat` y no con bytes `\x00` crudos.

### 4.5 Reglas de seguridad

1. Nunca confíes en `$args`: validá cantidad (`count($args)`), formato (`is_numeric`,
   `ctype_digit`) y pertenencia a un conjunto permitido (`in_array` con lista blanca)
   antes de usar cualquier valor.
2. Los valores leídos de `Config` llegan sin garantías de tipo: sanealos y aplicá
   rangos mínimos/máximos antes de usarlos en loops, timers o límites numéricos.
3. Todo comando declara permiso en `plugin.yml` y lo revisa con `testPermission()` o
   `hasPermission()` antes de ejecutar lógica.
4. Si hay SQL, entrá con prepared statements; prohibido armar el string de la query
   concatenando input.
5. Si hay rutas de archivos, construilas dentro de `getDataFolder()` con
   `basename()` sobre cualquier input del usuario: nada de `../` ni rutas absolutas.
6. No le muestres a los jugadores mensajes de error internos, stack traces ni rutas del
   sistema: logueá en el logger del plugin y devolvé un mensaje genérico.
7. Limpia los códigos de color del input de usuario antes de reenviarlo (chat, signos,
   libros) con `TextFormat::RESET` para evitar inyección de formato.
8. Si hay que hashear credenciales, usá `password_hash()` / `password_verify()`
   (disponibles en 7.0); nunca MD5 ni SHA1 crudos.
9. No guardes secretos ni tokens en `plugin.yml` ni en el código fuente.

## 5. Reglas de comportamiento del agente

1. **Este documento > tu memoria.** Ante cualquier duda de API, preferí el estilo PM2
   documentado acá antes que tu "recuerdo" de PM3/4/5.
2. **PHP 7.0.14 exacto.** Si al revisar tu propia salida detectás sintaxis 7.1+
   (`void`, `?tipo`, `fn`, `match`, `?->`, `??=`...), reescribila antes de entregar.
   Nunca entregues código "probado en PHP 8".
3. **Cero Composer.** No generes `composer.json`, `composer.lock` ni dependencias de
   Packagist: el flujo de la época es `plugin.yml` + phar/DevTools. Si una biblioteca
   es imprescindible, incluí su fuente dentro del plugin con su licencia.
4. **plugin.yml siempre.** Toda entrega de código PHP viene acompañada de su
   `plugin.yml` completo (`name`, `version`, `main`, `api: [2.0.0]`, más `commands` y
   `permissions` si aplica).
5. **No inventes APIs.** Si un detalle no está en este documento y no lo recordás con
   certeza, escribí tu mejor recuerdo y marcalo con `[VERIFICAR]` en un comentario para
   que el mantenedor lo chequeé; no frenes la entrega por eso.
6. **Snippets compilables.** Todo ejemplo marcado como correcto debe compilar en
   PHP 7.0.14: revisalo mentalmente línea por línea antes de entregarlo.
7. **Sin atajos modernos.** Nada de IDs string de items, clases `Vanilla*`, helpers de
   PHP 8 ni funciones que "reemplazan" código de la era con menos líneas.
8. **Strings del stack sagrado.** Si el usuario pide una API que no existe en 0.15.10
   (mobs, bloques o comandos de versiones posteriores), explicá la limitación y ofrecé
   la alternativa compatible con la era en lugar de forzar la API moderna.
9. **Cierre obligatorio.** Antes de terminar tu respuesta, recorré el SELF-CHECK de la
   sección 6 y corregí todo lo que falle.
10. **Sincronía de idiomas.** Si editás este archivo, aplicá el mismo cambio a
    [../en/AGENTS.md](../en/AGENTS.md).

## 6. SELF-CHECK (checklist obligatoria antes de entregar)

Marcá cada ítem. Si uno falla, no entregues: corregí y volvé a pasar la lista.

- [ ] Ningún `: void`, `?tipo`, `iterable` ni `catch (A | B $e)` en el código (7.1+).
- [ ] Ningún `[$a, $b] =`, ningún `list()` con claves, ningún `public const`, ningún `texto[-1]` (7.1+).
- [ ] Ninguna propiedad tipada, ningún `fn(`, ningún `??=`, ningún spread `[...`, ningún `object` (7.2-7.4).
- [ ] Ningún `match (`, argumento con nombre, constructor con propiedades promovidas, `#[`, `enum`, `readonly`, `mixed`, `never` ni union types (8.x).
- [ ] Ningún `str_contains(`, `str_starts_with(`, `str_ends_with(`, `get_debug_type(`, `?->` ni `throw` como expresión (8.x).
- [ ] Cero namespaces modernos de PocketMine (`pocketmine\player\`, `pocketmine\world\`) ni clases `World`, `VanillaItems`, `VanillaBlocks`, `ItemTypeIds`.
- [ ] Todo acceso a mundo usa `Level` y `$player->getLevel()`.
- [ ] Items y bloques usan IDs numéricos de la era 0.15; cero strings `"minecraft:..."`.
- [ ] `Task::onRun($currentTick)` no lleva tipo de retorno.
- [ ] No hay lógica en constructores: todo el arranque vive en `onEnable()`.
- [ ] No se generó `composer.json` ni dependencias de Packagist.
- [ ] El `plugin.yml` entregado está completo y declara `api: [2.0.0]`.
- [ ] El estado de jugadores usa `strtolower($player->getName())` como clave y no guarda objetos `Player`.
- [ ] Cero `eval()`, cero SQL concatenado, cero `sleep()` dentro del tick loop.
- [ ] Todo comando valida `$args` y revisa permisos antes de actuar.
- [ ] El código sigue PSR-2 (4 espacios, sin tabs, llaves según convención).
- [ ] Cada duda de API quedó marcada con `[VERIFICAR]` en un comentario.
- [ ] Si tocaste este archivo, actualizaste también [en/AGENTS.md](../en/AGENTS.md).
