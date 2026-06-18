
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Batalla de Pelotas</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #111827; /* Gray-900 */
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            user-select: none;
        }
        canvas {
            background-color: #1f2937; /* Gray-800 */
            box-shadow: 0 0 30px rgba(0,0,0,0.8);
            border: 4px solid #374151;
            border-radius: 12px;
            cursor: crosshair;
        }
        .ui-layer {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            pointer-events: none;
        }
        .menu-box {
            background-color: rgba(31, 41, 55, 0.95);
            padding: 2rem;
            border-radius: 1rem;
            border: 2px solid #4b5563;
            pointer-events: auto;
            min-width: 350px;
        }
        .test-sidebar {
            position: absolute;
            left: 10px;
            top: 50%;
            transform: translateY(-50%);
            background-color: rgba(31, 41, 55, 0.95);
            padding: 1rem;
            border-radius: 1rem;
            border: 2px solid #4b5563;
            display: flex;
            flex-direction: column;
            gap: 0.75rem;
            max-height: 80vh;
            overflow-y: auto;
            pointer-events: auto;
            width: 220px;
            z-index: 60;
        }
        select, button {
            pointer-events: auto;
        }
    </style>
</head>
<body>

    <!-- Lienzo del juego -->
    <canvas id="gameCanvas"></canvas>

    <!-- Botón Modo Prueba -->
    <button id="btnEnterTest" class="absolute top-4 right-4 bg-yellow-600 hover:bg-yellow-500 text-white font-bold py-2 px-4 rounded-lg transition-colors shadow-lg pointer-events-auto z-50">
        Modo Prueba 🛠️
    </button>

    <!-- Interfaz de Usuario superpuesta -->
    <div id="ui-layer" class="ui-layer">
        
        <!-- UI Modo Prueba (Barra Lateral Izquierda) -->
        <div id="testUI" class="hidden test-sidebar shadow-2xl">
            <h2 class="text-lg font-bold text-yellow-400 border-b border-gray-600 pb-2 mb-2 text-center">EDITOR</h2>
            <div class="flex flex-col gap-3">
                <label for="testBallType" class="text-xs font-semibold text-gray-300">Tipo de Pelota:</label>
                <select id="testBallType" class="bg-gray-800 text-white p-2 rounded border border-gray-600 text-xs focus:outline-none focus:border-yellow-500"></select>
                
                <button id="btnToggleDeleteMode" class="mt-2 bg-gray-600 hover:bg-red-500 text-white font-bold py-2 px-2 rounded text-xs transition-colors">Modo Eliminar: OFF</button>
                <button id="btnClearTest" class="bg-gray-700 hover:bg-gray-600 text-white font-bold py-2 px-2 rounded text-xs transition-colors">Limpiar Mapa</button>
                <button id="btnExitTest" class="mt-4 bg-blue-600 hover:bg-blue-500 text-white font-bold py-2 px-2 rounded text-xs transition-colors">Volver al Menú</button>
            </div>

            <div class="flex flex-col gap-2 mt-2 border-t border-gray-600 pt-2">
                <h3 class="text-[10px] font-bold text-gray-400 uppercase">Ajustes de Pelota</h3>
                <label for="testHPStep" class="text-xs font-semibold text-gray-300">Cant. Vida +/-:</label>
                <input id="testHPStep" type="number" value="10" class="bg-gray-800 text-white p-2 rounded border border-gray-600 text-xs focus:outline-none focus:border-yellow-500">
            </div>
        </div>

        <!-- Menú Principal -->
        <div id="menu" class="menu-box shadow-2xl flex flex-col gap-4">
            <h1 class="text-3xl font-bold text-center text-blue-400 mb-2">Batalla de Pelotas</h1>
            
            <div class="flex justify-between items-center bg-gray-700 p-3 rounded-lg">
                <label for="numBalls" class="font-semibold">Cantidad de Pelotas:</label>
                <select id="numBalls" class="bg-gray-800 text-white p-2 rounded border border-gray-600 focus:outline-none focus:border-blue-500">
                    <option value="2">2 Pelotas</option>
                    <option value="3">3 Pelotas</option>
                    <option value="4">4 Pelotas</option>
                </select>
            </div>

            <div class="flex flex-col bg-gray-700 p-3 rounded-lg gap-2">
                <label for="searchBall" class="font-semibold text-sm text-blue-300">Buscar Pelota:</label>
                <input id="searchBall" type="text" placeholder="Filtrar tipos..." class="bg-gray-800 text-white p-2 rounded border border-gray-600 focus:outline-none focus:border-blue-500 text-sm">
            </div>

            <div id="playerConfigs" class="flex flex-col gap-3 mt-2">
                <!-- Se llena dinámicamente con JS -->
            </div>

            <button id="btnStart" class="mt-4 bg-blue-600 hover:bg-blue-500 text-white font-bold py-3 px-6 rounded-lg transition-colors text-lg shadow-lg">
                Comenzar Juego
            </button>
            
            <div class="text-xs text-gray-400 mt-2">
  
            </div>
        </div>

        <!-- Mensajes en pantalla (Apuntando, Ganador, etc) -->
        <div id="messageOverlay" class="hidden absolute top-10 text-2xl font-bold bg-black/60 px-6 py-3 rounded-2xl text-white shadow-lg backdrop-blur-sm pointer-events-none transition-all whitespace-pre-wrap">
            Mensaje
        </div>

        <!-- Botón Reiniciar -->
        <button id="btnRestart" class="hidden absolute bottom-10 bg-green-600 hover:bg-green-500 text-white font-bold py-3 px-8 rounded-lg transition-colors text-xl shadow-lg pointer-events-auto">
            Volver al Menú
        </button>
        
        <!-- Botón Salir de Pelea -->
        <button id="btnExitFight" class="hidden absolute top-10 right-10 bg-red-600 hover:bg-red-500 text-white font-bold py-3 px-8 rounded-lg transition-colors text-xl shadow-lg pointer-events-auto">
            Salir de la Pelea
        </button>

        <!-- Botón Toggle Velocidad -->
        <button id="btnToggleSpeed" class="hidden absolute top-10 left-10 bg-purple-600 hover:bg-purple-500 text-white font-bold py-3 px-8 rounded-lg transition-colors text-xl shadow-lg pointer-events-auto">
            Velocidad x2
        </button>

        <!-- HUD de Vidas Lateral -->
        <div id="hud" class="hidden absolute right-10 top-1/2 -translate-y-1/2 flex flex-col gap-4 p-5 bg-gray-800/80 rounded-2xl border-2 border-gray-600 backdrop-blur-md pointer-events-auto min-w-[220px] z-50">
            <h2 class="text-xl font-bold text-blue-400 border-b border-gray-600 pb-2 mb-2 text-center uppercase tracking-wider">Equipos</h2>
            <div id="hudList" class="flex flex-col gap-4"></div>
        </div>

    
        <!-- Texto Donar -->
        <div class="absolute bottom-4 left-4 text-xs text-gray-400 pointer-events-none z-50">
            Si quieres donar este es mi alias de Mercado Pago: <span class="text-purple-400 font-bold">alan.358.abiamor.mp</span>
        </div>

    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const menu = document.getElementById('menu');
        const playerConfigsDiv = document.getElementById('playerConfigs');
        const numBallsSelect = document.getElementById('numBalls');
        const btnStart = document.getElementById('btnStart');
        const messageOverlay = document.getElementById('messageOverlay');
        const btnRestart = document.getElementById('btnRestart');
        const hud = document.getElementById('hud');
        const hudList = document.getElementById('hudList');
        const btnExitFight = document.getElementById('btnExitFight');
        const btnToggleSpeed = document.getElementById('btnToggleSpeed');
        const btnEnterTest = document.getElementById('btnEnterTest');
        const btnExitTest = document.getElementById('btnExitTest');
        const testUI = document.getElementById('testUI');
        const testBallTypeSelect = document.getElementById('testBallType');
        const btnToggleDeleteMode = document.getElementById('btnToggleDeleteMode');
        const btnClearTest = document.getElementById('btnClearTest');

        // Configuración general
        const CANVAS_SIZE = Math.min(window.innerWidth, window.innerHeight) * 0.85;
        canvas.width = CANVAS_SIZE;
        canvas.height = CANVAS_SIZE;
        
        const BASE_SPEED = 4; // Velocidad base de las pelotas
        const DASH_SPEED = 12; // Velocidad de dash de las pelotas rápidas
        const BALL_RADIUS = 45;
        const EXPLOSION_RADIUS_MULTIPLIER = 3;
        const EXPLOSION_EFFECT_DURATION = 70;

        const PROYECTILE_RADIUS = 5;
        const PROYECTILE_SPEED = 8;
        
        // Estados del juego
        const STATE_MENU = 0;
        const STATE_AIMING = 1;
        const STATE_PLAYING = 2;
        const STATE_GAMEOVER = 3;
        const STATE_TEST = 4;
        
        let gameState = STATE_MENU;
        let balls = [];
        let projectiles = [];
        let discs = [];
        let floatingTexts = [];
        let radioactiveTrails = []; 
        let vines = []; 
        let toxicZones = []; 
        let explosionEffects = []; 
        let waterPuddles = []; 
        let fireballs = []; 
        let closedSpaces = []; 
        let sonicWaves = []; 
        let apples = []; 
        let boomerangs = []; 
        let orbitalPlanets = []; 
        let dongEffects = []; 
        let cacti = []; 
        let laserBeamsV2 = []; 
        let woodenPlanks = []; 
        let crystalShards = []; 
        let darkSouls = []; 
        let ranking = []; 
        let spiderThreads = []; 
        let codeLines = []; 
        let sandiaSeeds = [];
        let sandiaProjectiles = [];
        let drones = []; 
        let meteorites = [];
        let blackBalls = [];
        let lavaVolcanoes = [];
        let spiritualSouls = [];

        let selectedBall = null;
        let gameSpeedMultiplier = 1; 
        let deleteModeActive = false;
        let modifyingBallId = null;

        // Variables para interacción de apuntado
        let isDragging = false;
        let dragStart = { x: 0, y: 0 };
        let dragCurrent = { x: 0, y: 0 };

        // Tipos de pelotas
        const TYPE_NORMAL = 'normal';
        const TYPE_ARMADA = 'armada';
        const TYPE_RAPIDA = 'rapida';
        const TYPE_VAMPIRO = 'vampiro';
        const TYPE_EXPLOSIVA = 'explosiva';
        const TYPE_BOMBA = 'bomba';
        const TYPE_TOXICA = 'toxica';
        const TYPE_LASER = 'laser';
        const TYPE_ESPADACHINA = 'espadachina';
        const TYPE_RADIOACTIVA = 'radioactiva';
        const TYPE_CERRADA = 'cerrada';
        const TYPE_MAGNETICA = 'magnetica';
        const TYPE_CLONADORA = 'clonadora';
        const TYPE_FANTASMA = 'fantasma';
        const TYPE_ELECTRICA = 'electrica';
        const TYPE_AGUA = 'agua';
        const TYPE_FUEGO = 'fuego';
        const TYPE_CARNIVORA = 'carnivora';
        const TYPE_PLANTADORA = 'plantadora';
        const TYPE_SONORA = 'sonora';
        const TYPE_GELATINA = 'gelatina';
        const TYPE_HIELO = 'hielo';
        const TYPE_ALEATORIA = 'aleatoria';
        const TYPE_MANZANA = 'manzana';
        const TYPE_BOOMERANG = 'boomerang';
        const TYPE_ORBITAL = 'orbital';
        const TYPE_PORTAL = 'portal';
        const TYPE_CAMPANA = 'campana';
        const TYPE_BICHO = 'bicho';
        const TYPE_DESERTICA = 'desertica';
        const TYPE_LASER_V2 = 'laser_v2';
        const TYPE_ESCUDERA = 'escudera';
        const TYPE_MADERA = 'madera';
        const TYPE_APOSTADORA = 'apostadora';
        const TYPE_DESLIZANTE = 'deslizante';
        const TYPE_METALICA = 'metalica';
        const TYPE_AMETRALLADORA = 'ametralladora';
        const TYPE_CANON = 'canon';
        const TYPE_CRISTAL = 'cristal';
        const TYPE_MUERTE = 'muerte';
        const TYPE_FURIA = 'furia';
        const TYPE_ARANA = 'arana';
        const TYPE_PROGRAMADORA = 'programadora';
        const TYPE_DRON = 'dron';
        const TYPE_TOPO = 'topo';
        const TYPE_ERROR = 'error';
        const TYPE_GRAVITACIONAL = 'gravitacional';
        const TYPE_DIVISORA = 'divisora';
        const TYPE_SANDIA = 'sandia';
        const TYPE_TIEMPO = 'tiempo';
        const TYPE_GOLEM = 'golem';
        const TYPE_DOBLE = 'doble';
        const TYPE_RAYO = 'rayo';
        const TYPE_PIEDRA = 'piedra';
        const TYPE_DIMENSIONAL = 'dimensional';
        const TYPE_CONEJO = 'conejo';
        const TYPE_NUMERICA = 'numerica';
        const TYPE_PROYECCION = 'proyeccion';
        const TYPE_PRUEBA = 'prueba';
        const TYPE_DISCO = 'disco';
        const TYPE_NUCLEAR = 'nuclear';
        const TYPE_HOMBRELOBO = 'hombrelobo';
        const TYPE_ESPACIO = 'espacio';
        const TYPE_LAVA = 'lava';
        const TYPE_ESPIRITUAL = 'espiritual';
        const TYPE_CUESTIONADORA = 'cuestionadora';
        const TYPE_ABSORVEDORA = 'absorvedora';
        const TYPE_SOL = 'sol';
        const TYPE_COLOR = 'color';
        const TYPE_COMPRA_VENTA = 'compra_venta';

        const COLORS = {
            [TYPE_NORMAL]: '#ffffff',
            [TYPE_ARMADA]: '#fde047',
            [TYPE_RAPIDA]: '#fca5a5',
            [TYPE_VAMPIRO]: '#8b0000',
            [TYPE_EXPLOSIVA]: '#ffa500',
            [TYPE_BOMBA]: '#4b5563',
            [TYPE_TOXICA]: '#90ee90',
            [TYPE_LASER]: '#ff8c00',
            [TYPE_ESPADACHINA]: '#d3d3d3',
            [TYPE_RADIOACTIVA]: '#39ff14',
            [TYPE_CERRADA]: '#9333ea',
            [TYPE_MAGNETICA]: '#d946ef',
            [TYPE_CLONADORA]: '#60a5fa',
            [TYPE_FANTASMA]: '#e0f2fe',
            [TYPE_ELECTRICA]: '#fef08a',
            [TYPE_AGUA]: '#0ea5e9',
            [TYPE_FUEGO]: '#f97316',
            [TYPE_CARNIVORA]: '#be185d',
            [TYPE_PLANTADORA]: '#22c55e',
            [TYPE_SONORA]: '#e0f2fe',
            [TYPE_GELATINA]: '#f472b6',
            [TYPE_HIELO]: '#a5f3fc',
            [TYPE_ALEATORIA]: '#a855f7',
            [TYPE_MANZANA]: '#ff4d4d',
            [TYPE_BOOMERANG]: '#94a3b8',
            [TYPE_ORBITAL]: '#fbbf24',
            [TYPE_PORTAL]: '#8b5cf6',
            [TYPE_CAMPANA]: '#fde68a',
            [TYPE_BICHO]: '#65a30d',
            [TYPE_DESERTICA]: '#c2b280',
            [TYPE_LASER_V2]: '#06b6d4',
            [TYPE_ESCUDERA]: '#94a3b8',
            [TYPE_MADERA]: '#d97706',
            [TYPE_APOSTADORA]: '#fbbf24',
            [TYPE_DESLIZANTE]: '#2dd4bf',
            [TYPE_METALICA]: '#cbd5e1',
            [TYPE_AMETRALLADORA]: '#64748b',
            [TYPE_CANON]: '#1f2937',
            [TYPE_CRISTAL]: '#f1f5f9',
            [TYPE_MUERTE]: '#000000',
            [TYPE_FURIA]: '#ef4444',
            [TYPE_ARANA]: '#451a03',
            [TYPE_DRON]: '#93c5fd',
            [TYPE_TOPO]: '#78350f',
            [TYPE_ERROR]: '#d946ef',
            [TYPE_GRAVITACIONAL]: '#38bdf8',
            [TYPE_DIVISORA]: '#ef4444',
            [TYPE_SANDIA]: '#4ade80',
            [TYPE_TIEMPO]: '#6366f1',
            [TYPE_GOLEM]: '#78716c',
            [TYPE_DOBLE]: '#7dd3fc',
            [TYPE_RAYO]: '#fde047',
            [TYPE_PIEDRA]: '#a8a29e',
            [TYPE_DIMENSIONAL]: '#6d28d9', /* Violet-700 */
            [TYPE_CONEJO]: '#fdf2f8',
            [TYPE_NUMERICA]: '#06b6d4',
            [TYPE_PROYECCION]: '#818cf8',
            [TYPE_PRUEBA]: '#fb7185',
            [TYPE_DISCO]: '#c084fc',
            [TYPE_NUCLEAR]: '#4ade80',
            [TYPE_HOMBRELOBO]: '#94a3b8',
            [TYPE_ESPACIO]: '#1e1b4b',
            [TYPE_LAVA]: '#f97316',
            [TYPE_ESPIRITUAL]: '#e0e7ff',
            [TYPE_CUESTIONADORA]: '#6366f1',
            [TYPE_ABSORVEDORA]: '#10b981',
            [TYPE_SOL]: '#facc15',
            [TYPE_COLOR]: '#ffffff',
            [TYPE_COMPRA_VENTA]: '#22c55e',
        };

        const BORDER_COLORS = {
            [TYPE_NORMAL]: '#9ca3af',
            [TYPE_ARMADA]: '#ca8a04',
            [TYPE_RAPIDA]: '#b91c1c',
            [TYPE_VAMPIRO]: '#5a0000',
            [TYPE_EXPLOSIVA]: '#cc8400',
            [TYPE_BOMBA]: '#111827',
            [TYPE_TOXICA]: '#5cb85c',
            [TYPE_LASER]: '#cc7000',
            [TYPE_ESPADACHINA]: '#a9a9a9',
            [TYPE_RADIOACTIVA]: '#00e600',
            [TYPE_CERRADA]: '#581c87',
            [TYPE_MAGNETICA]: '#701a75',
            [TYPE_CLONADORA]: '#2563eb',
            [TYPE_FANTASMA]: '#7dd3fc',
            [TYPE_ELECTRICA]: '#eab308',
            [TYPE_AGUA]: '#0284c7',
            [TYPE_FUEGO]: '#9a3412',
            [TYPE_CARNIVORA]: '#701a75',
            [TYPE_PLANTADORA]: '#166534',
            [TYPE_SONORA]: '#38bdf8',
            [TYPE_GELATINA]: '#be185d',
            [TYPE_HIELO]: '#0891b2',
            [TYPE_ALEATORIA]: '#6b21a8',
            [TYPE_MANZANA]: '#b30000',
            [TYPE_BOOMERANG]: '#475569',
            [TYPE_ORBITAL]: '#92400e',
            [TYPE_PORTAL]: '#4c1d95',
            [TYPE_CAMPANA]: '#b45309',
            [TYPE_BICHO]: '#365314',
            [TYPE_DESERTICA]: '#8b7355',
            [TYPE_LASER_V2]: '#0891b2',
            [TYPE_ESCUDERA]: '#334155',
            [TYPE_MADERA]: '#78350f',
            [TYPE_APOSTADORA]: '#b45309',
            [TYPE_DESLIZANTE]: '#0f766e',
            [TYPE_METALICA]: '#475569',
            [TYPE_AMETRALLADORA]: '#334155',
            [TYPE_CANON]: '#000000',
            [TYPE_CRISTAL]: '#94a3b8',
            [TYPE_MUERTE]: '#4c1d95',
            [TYPE_FURIA]: '#7f1d1d',
            [TYPE_ARANA]: '#1c1917',
            [TYPE_DRON]: '#1e40af',
            [TYPE_TOPO]: '#451a03',
            [TYPE_ERROR]: '#701a75',
            [TYPE_GRAVITACIONAL]: '#0369a1',
            [TYPE_DIVISORA]: '#991b1b',
            [TYPE_SANDIA]: '#14532d',
            [TYPE_TIEMPO]: '#312e81',
            [TYPE_GOLEM]: '#44403c',
            [TYPE_DOBLE]: '#0ea5e9',
            [TYPE_RAYO]: '#b45309',
            [TYPE_PIEDRA]: '#78716c',
            [TYPE_DIMENSIONAL]: '#a78bfa', /* Violet-400 */
            [TYPE_CONEJO]: '#ec4899',
            [TYPE_NUMERICA]: '#0891b2',
            [TYPE_PROYECCION]: '#4f46e5',
            [TYPE_PRUEBA]: '#e11d48',
            [TYPE_DISCO]: '#7e22ce',
            [TYPE_NUCLEAR]: '#14532d',
            [TYPE_HOMBRELOBO]: '#475569',
            [TYPE_ESPACIO]: '#4338ca',
            [TYPE_LAVA]: '#7c2d12',
            [TYPE_ESPIRITUAL]: '#4f46e5',
            [TYPE_CUESTIONADORA]: '#4338ca',
            [TYPE_ABSORVEDORA]: '#064e3b',
            [TYPE_SOL]: '#ea580c',
            [TYPE_COLOR]: '#1f2937',
            [TYPE_COMPRA_VENTA]: '#166534',
        };

        const ALL_BALL_TYPES = [
            { value: 'random_pick', label: "Pelota" },
            { value: TYPE_NORMAL, label: "Pelota Normal" },
            { value: TYPE_ARMADA, label: "Pelota Armada" },
            { value: TYPE_RAPIDA, label: "Pelota Rápida" },
            { value: TYPE_VAMPIRO, label: "Pelota Vampiro" },
            { value: TYPE_EXPLOSIVA, label: "Pelota Explosiva" },
            { value: TYPE_BOMBA, label: "Pelota Bomba" },
            { value: TYPE_TOXICA, label: "Pelota Tóxica" },
            { value: TYPE_LASER, label: "Pelota Láser" },
            { value: TYPE_ESPADACHINA, label: "Pelota Espadachina" },
            { value: TYPE_RADIOACTIVA, label: "Pelota Radioactiva" },
            { value: TYPE_CERRADA, label: "Pelota Cerrada" },
            { value: TYPE_MAGNETICA, label: "Pelota Magnética" },
            { value: TYPE_CLONADORA, label: "Pelota Clonadora" },
            { value: TYPE_FANTASMA, label: "Pelota Fantasma" },
            { value: TYPE_ELECTRICA, label: "Pelota Eléctrica" },
            { value: TYPE_AGUA, label: "Pelota de Agua" },
            { value: TYPE_FUEGO, label: "Pelota de Fuego" },
            { value: TYPE_CARNIVORA, label: "Pelota Carnívora" },
            { value: TYPE_PLANTADORA, label: "Pelota Plantadora" },
            { value: TYPE_SONORA, label: "Pelota Sonora" },
            { value: TYPE_GELATINA, label: "Pelota Gelatina" },
            { value: TYPE_HIELO, label: "Pelota Hielo" },
            { value: TYPE_ALEATORIA, label: "Pelota Aleatoria" },
            { value: TYPE_MANZANA, label: "Pelota Manzana" },
            { value: TYPE_BOOMERANG, label: "Pelota Boomerang" },
            { value: TYPE_ORBITAL, label: "Pelota Orbital" },
            { value: TYPE_PORTAL, label: "Pelota Portal" },
            { value: TYPE_CAMPANA, label: "Pelota Campana" },
            { value: TYPE_BICHO, label: "Pelota Bicho" },
            { value: TYPE_DESERTICA, label: "Pelota Desértica" },
            { value: TYPE_LASER_V2, label: "Pelota Láser V2" },
            { value: TYPE_ESCUDERA, label: "Pelota Escudera" },
            { value: TYPE_MADERA, label: "Pelota de Madera" },
            { value: TYPE_APOSTADORA, label: "Pelota Apostadora" },
            { value: TYPE_DESLIZANTE, label: "Pelota Deslizante" },
            { value: TYPE_METALICA, label: "Pelota Metálica" },
            { value: TYPE_AMETRALLADORA, label: "Pelota Ametralladora" },
            { value: TYPE_CANON, label: "Pelota Cañón" },
            { value: TYPE_CRISTAL, label: "Pelota de Cristal" },
            { value: TYPE_MUERTE, label: "Pelota de la Muerte" },
            { value: TYPE_FURIA, label: "Pelota Furia" },
            { value: TYPE_ARANA, label: "Pelota Araña" },
            { value: TYPE_PROGRAMADORA, label: "Pelota Programadora" },
            { value: TYPE_DRON, label: "Pelota Dron" },
            { value: TYPE_TOPO, label: "Pelota Topo" },
            { value: TYPE_GRAVITACIONAL, label: "Pelota Gravitacional" },
            { value: TYPE_DIVISORA, label: "Pelota Divisora" },
            { value: TYPE_SANDIA, label: "Pelota Sandía" },
            { value: TYPE_TIEMPO, label: "Pelota de Tiempo" },
            { value: TYPE_GOLEM, label: "Pelota Golem" },
            { value: TYPE_DOBLE, label: "Pelota Doble" },
            { value: TYPE_RAYO, label: "Pelota Rayo" },
            { value: TYPE_PIEDRA, label: "Pelota de Piedra" },
            { value: TYPE_DIMENSIONAL, label: "Pelota Dimensional" },
            { value: TYPE_CONEJO, label: "Pelota Conejo" },
            { value: TYPE_NUMERICA, label: "Pelota Numérica" },
            { value: TYPE_PROYECCION, label: "Pelota de Proyección" },
            { value: TYPE_ERROR, label: "Pelota Error" },
            { value: TYPE_DISCO, label: "Pelota Disco" },
            { value: TYPE_NUCLEAR, label: "Pelota Nuclear" },
            { value: TYPE_HOMBRELOBO, label: "Pelota Hombre Lobo" },
            { value: TYPE_ESPACIO, label: "Pelota Espacio" },
            { value: TYPE_LAVA, label: "Pelota de Lava" },
            { value: TYPE_ESPIRITUAL, label: "Pelota Espiritual" },
            { value: TYPE_CUESTIONADORA, label: "Pelota Cuestionadora" },
            { value: TYPE_ABSORVEDORA, label: "Pelota Absorbedora" },
            { value: TYPE_SOL, label: "Pelota Sol" },
            { value: TYPE_COLOR, label: "Pelota Color" },
        ];

        class FloatingText {
            constructor(x, y, text, color) {
                this.x = x;
                this.y = y;
                this.text = text;
                this.color = color;
                this.life = 60; // 1 segundo a 60fps
                this.maxLife = 60;
            }
            update() {
                this.y -= 1;
                this.life--;
            }
            draw(ctx) {
                const alpha = this.life / this.maxLife;
                ctx.globalAlpha = alpha;
                ctx.fillStyle = this.color;
                ctx.font = 'bold 20px Arial';
                ctx.textAlign = 'center';
                ctx.fillText(this.text, this.x, this.y);
                ctx.globalAlpha = 1.0;
            }
        }

        class SpiritualSoul {
            constructor(x, y, vx, vy, ownerId) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.ownerId = ownerId;
                this.radius = 10;
                this.active = true;
                this.damage = 5;
                this.damage = 4;
                this.hitCooldowns = new Map();
            }

            update(ballsArray) {
                this.x += this.vx;
                this.y += this.vy;
                if (this.x < -100 || this.x > canvas.width + 100 || this.y < -100 || this.y > canvas.height + 100) {
                    this.active = false;
                    return;
                }
                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId) {
                        if (Math.hypot(this.x - b.x, this.y - b.y) < this.radius + b.radius) {
                            let cd = this.hitCooldowns.get(b.id) || 0;
                            if (cd <= 0) {
                                b.takeDamage(this.damage, null, false, false, true);
                                this.hitCooldowns.set(b.id, 60);
                            }
                        }
                    }
                });
                this.hitCooldowns.forEach((val, key) => { if (val > 0) this.hitCooldowns.set(key, val - 1); });
            }

            draw(ctx) {
                ctx.save();
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                const grad = ctx.createRadialGradient(this.x, this.y, 0, this.x, this.y, this.radius * 2);
                grad.addColorStop(0, 'rgba(255, 255, 255, 0.9)');
                grad.addColorStop(1, 'rgba(129, 140, 248, 0)');
                ctx.fillStyle = grad;
                ctx.fill();
                ctx.restore();
            }
        }

        class BlackBall {
            constructor(x, y, vx, vy, ownerId) {
                this.x = x; this.y = y; this.vx = vx; this.vy = vy;
                this.ownerId = ownerId; this.radius = BALL_RADIUS * 0.5;
                this.active = true;
                this.hitCooldowns = new Map();
            }
            update(ballsArray) {
                this.x += this.vx; this.y += this.vy;
                if (this.x < this.radius || this.x > canvas.width - this.radius) this.vx *= -1;
                if (this.y < this.radius || this.y > canvas.height - this.radius) this.vy *= -1;

                ballsArray.forEach(b => {
                    if (b.isAlive) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            let cd = this.hitCooldowns.get(b.id) || 0;
                            if (cd <= 0) {
                                if (b.id === this.ownerId) {
                                    b.takeDamage(1, null, false, false, true);
                                } else {
                                    b.takeDamage(2);
                                }
                                this.hitCooldowns.set(b.id, 45);
                            }
                        }
                    }
                    if (this.hitCooldowns.has(b.id)) {
                        let val = this.hitCooldowns.get(b.id);
                        if (val > 0) this.hitCooldowns.set(b.id, val - 1);
                    }
                });
            }
            draw(ctx) {
                ctx.save();
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(0, 0, 0, 0.4)';
                ctx.fill();
                ctx.strokeStyle = 'rgba(255, 255, 255, 0.2)';
                ctx.lineWidth = 1;
                ctx.stroke();
                ctx.closePath();
                ctx.restore();
            }
        }

        class SandiaSeed {
            constructor(x, y, vx, vy, ownerId, damage, bounces = 3) {
                this.x = x; this.y = y; this.vx = vx; this.vy = vy;
                this.ownerId = ownerId; this.damage = damage;
                this.radius = 4; this.bounces = bounces; this.active = true;
            }
            update(ballsArray) {
                this.x += this.vx; this.y += this.vy;
                let hitWall = false;
                if (this.x < 0 || this.x > canvas.width) { this.vx *= -1; hitWall = true; }
                if (this.y < 0 || this.y > canvas.height) { this.vy *= -1; hitWall = true; }
                if (hitWall) { this.bounces--; if (this.bounces < 0) this.active = false; }

                for (let b of ballsArray) {
                    if (b.isAlive && b.id !== this.ownerId) {
                        if (Math.hypot(this.x - b.x, this.y - b.y) < b.radius + this.radius) {
                            b.takeDamage(this.damage);
                            if (this.damage === 2) {
                                const ang = Math.random() * Math.PI * 2;
                                const speed = getEffectiveSpeed(6);
                                sandiaProjectiles.push(new SandiaProjectile(this.x, this.y, Math.cos(ang) * speed, Math.sin(ang) * speed, this.ownerId));
                            }
                            this.active = false; break;
                        }
                    }
                }
            }
            draw(ctx) {
                ctx.beginPath(); ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#111827'; ctx.fill(); ctx.closePath();
            }
        }

        class SandiaProjectile {
            constructor(x, y, vx, vy, ownerId) {
                this.x = x; this.y = y; this.vx = vx; this.vy = vy;
                this.ownerId = ownerId; this.radius = 12; this.bounces = 3; this.active = true;
                this.spawnTimer = 30; // 0.5 segundos a 60fps
            }
            update(ballsArray) {
                this.x += this.vx; this.y += this.vy;
                let hitWall = false;
                if (this.spawnTimer > 0) this.spawnTimer--;
                if (this.x < this.radius || this.x > canvas.width - this.radius) { 
                    this.x = this.x < this.radius ? this.radius : canvas.width - this.radius;
                    this.vx *= -1; hitWall = true; 
                }
                if (this.y < this.radius || this.y > canvas.height - this.radius) { 
                    this.y = this.y < this.radius ? this.radius : canvas.height - this.radius;
                    this.vy *= -1; hitWall = true; 
                }
                if (hitWall) {
                    this.bounces--;
                    for (let i = 0; i < 2; i++) {
                        let ang = Math.random() * Math.PI * 2;
                        sandiaSeeds.push(new SandiaSeed(this.x, this.y, Math.cos(ang) * 5, Math.sin(ang) * 5, this.ownerId, 1));
                    }
                    if (this.bounces < 0) this.active = false;
                }
                if (this.spawnTimer <= 0) {
                    for (let b of ballsArray) {
                        if (b.isAlive && b.id !== this.ownerId) {
                            if (Math.hypot(this.x - b.x, this.y - b.y) < b.radius + this.radius) {
                                b.takeDamage(3);
                                this.active = false; break;
                            }
                        }
                    }
                }
            }
            draw(ctx) {
                ctx.beginPath(); ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#22c55e'; ctx.fill();
                if (this.spawnTimer > 0) ctx.globalAlpha = 0.5; // Efecto visual mientras no hace daño
                ctx.strokeStyle = '#166534'; ctx.lineWidth = 2; ctx.stroke();
                ctx.fillStyle = '#111827'; ctx.beginPath(); ctx.arc(this.x-3, this.y-2, 2, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.arc(this.x+3, this.y+2, 2, 0, Math.PI*2); ctx.fill(); ctx.closePath();
            }
        }

        class Meteorite {
          
            constructor(x, y, vx, vy, ownerId) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.ownerId = ownerId;
                this.radius = BALL_RADIUS * 0.4; // 60% más pequeño
                this.active = true;
                this.damage = 1;
            }

            update(ballsArray) {
                this.x += this.vx;
                this.y += this.vy;
                if (this.x < -100 || this.x > canvas.width + 100 || this.y < -100 || this.y > canvas.height + 100) {
                    this.active = false;
                }
                for (let b of ballsArray) {
                    if (b.isAlive && b.id !== this.ownerId && Math.hypot(this.x - b.x, this.y - b.y) < b.radius + this.radius) {
                        b.takeDamage(this.damage);
                        this.active = false;
                        break;
                    }
                }
            }

            draw(ctx) {
                ctx.save();
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#4b5563';
                ctx.fill();
                ctx.strokeStyle = '#1f2937';
                ctx.lineWidth = 2;
                ctx.stroke();
                ctx.restore();
            }
        }

        class Volcano {
            constructor(x, y, side, ownerId) {
                this.x = x;
                this.y = y;
                this.side = side;
                this.ownerId = ownerId;
                this.radius = 35;
                this.radius = 31.5;
                this.eruptionTimer = 30; // 0.5 segundos para erupcionar
                this.active = true;
                this.isErupting = false;
                this.eruptionLife = 150; // 2.5 segundos de duración (1s + 1.5s extra)
                this.eruptionLife = 120; // 2 segundos de duración
                this.postEruptionLife = 130; // 2 segundos de permanencia tras el rayo
                this.damageCooldowns = new Map();
            }

            update(ballsArray) {
                if (this.eruptionTimer > 0) {
                    this.eruptionTimer--;
                    if (this.eruptionTimer <= 0) this.isErupting = true;
                } else if (this.isErupting) {
                    this.eruptionLife--;
                    
                    // Daño de la línea roja
                    ballsArray.forEach(b => {
                        if (b.isAlive && b.id !== this.ownerId) {
                            let hitLine = false;
                            if (this.side === 'left' || this.side === 'right') {
                                if (Math.abs(b.y - this.y) < b.radius + 10) hitLine = true;
                                if (Math.abs(b.y - this.y) < b.radius + 9) hitLine = true;
                            } else {
                                if (Math.abs(b.x - this.x) < b.radius + 10) hitLine = true;
                                if (Math.abs(b.x - this.x) < b.radius + 9) hitLine = true;
                            }

                            if (hitLine) {
                                let cd = this.damageCooldowns.get(b.id + "_line") || 0;
                                if (cd <= 0) {
                                    b.takeDamage(1);
                                    this.damageCooldowns.set(b.id + "_line", 30);
                                }
                            }
                        }
                    });

                    if (this.eruptionLife <= 0) this.active = false;
                    if (this.eruptionLife <= 0) this.isErupting = false;
                } else {
                    // Fase post-erupción (el volcán sigue ahí pero sin rayo)
                    this.postEruptionLife--;
                    if (this.postEruptionLife <= 0) this.active = false;
                }

                // Daño por contacto con el volcán (4 de daño cada 0.5s)
                // Daño por contacto con el volcán (4 de daño por segundo)
                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId) {
                        if (Math.hypot(this.x - b.x, this.y - b.y) < this.radius + b.radius) {
                            let cd = this.damageCooldowns.get(b.id + "_contact") || 0;
                            if (cd <= 0) {
                                b.takeDamage(4);
                                this.damageCooldowns.set(b.id + "_contact", 30);
                                this.damageCooldowns.set(b.id + "_contact", 60);
                            }
                        }
                    }
                });

                this.damageCooldowns.forEach((val, key) => {
                    if (val > 0) this.damageCooldowns.set(key, val - 1);
                });
            }

            draw(ctx) {
                ctx.save();
                ctx.fillStyle = this.isErupting ? '#ef4444' : '#7c2d12';
                ctx.beginPath();
                if (this.side === 'left') { ctx.moveTo(0, this.y - this.radius); ctx.lineTo(this.radius, this.y); ctx.lineTo(0, this.y + this.radius); }
                else if (this.side === 'right') { ctx.moveTo(canvas.width, this.y - this.radius); ctx.lineTo(canvas.width - this.radius, this.y); ctx.lineTo(canvas.width, this.y + this.radius); }
                else if (this.side === 'top') { ctx.moveTo(this.x - this.radius, 0); ctx.lineTo(this.x, this.radius); ctx.lineTo(this.x + this.radius, 0); }
                else { ctx.moveTo(this.x - this.radius, canvas.height); ctx.lineTo(this.x, canvas.height - this.radius); ctx.lineTo(this.x + this.radius, canvas.height); }
                ctx.fill();

                if (this.isErupting) {
                    ctx.strokeStyle = 'rgba(239, 68, 68, 0.8)';
                    ctx.lineWidth = 15;
                    ctx.lineWidth = 13.5;
                    ctx.beginPath();
                    if (this.side === 'left' || this.side === 'right') { ctx.moveTo(0, this.y); ctx.lineTo(canvas.width, this.y); }
                    else { ctx.moveTo(this.x, 0); ctx.lineTo(this.x, canvas.height); }
                    ctx.stroke();
                }
                ctx.restore();
            }
        }

        class Projectile {
            constructor(x, y, vx, vy, ownerId, damage = 2, radius = PROYECTILE_RADIUS, color = '#f59e0b') {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.ownerId = ownerId;
                this.radius = radius;
                this.active = true;
                this.damage = damage;
                this.color = color;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                if (this.x < 0 || this.x > canvas.width || this.y < 0 || this.y > canvas.height) {
                    this.active = false;
                }
            }

            draw(ctx) {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();
            }
        }

        class Disc {
            constructor(x, y, vx, vy, ownerId) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.ownerId = ownerId;
                this.radius = BALL_RADIUS * 1.2;
                this.bounces = 4;
                this.active = true;
                this.angle = 0;
                this.rotationSpeed = 0.25;
                this.hitCooldowns = new Map();
            }

            update(ballsArray) {
                this.x += this.vx;
                this.y += this.vy;
                this.angle += this.rotationSpeed;

                let hitWall = false;
                if (this.x - this.radius < 0) { this.x = this.radius; this.vx *= -1; hitWall = true; }
                if (this.x + this.radius > canvas.width) { this.x = canvas.width - this.radius; this.vx *= -1; hitWall = true; }
                if (this.y - this.radius < 0) { this.y = this.radius; this.vy *= -1; hitWall = true; }
                if (this.y + this.radius > canvas.height) { this.y = canvas.height - this.radius; this.vy *= -1; hitWall = true; }

                if (hitWall) {
                    this.bounces--;
                    if (this.bounces < 0) this.active = false;
                }

                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            let cd = this.hitCooldowns.get(b.id) || 0;
                            if (cd <= 0) {
                                b.takeDamage(7);
                                b.takeDamage(5);
                                this.hitCooldowns.set(b.id, 25);
                            }
                        }
                    }
                    if (this.hitCooldowns.has(b.id)) {
                        let current = this.hitCooldowns.get(b.id);
                        if (current > 0) this.hitCooldowns.set(b.id, current - 1);
                    }
                });
            }

            draw(ctx) {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);
                ctx.beginPath();
                ctx.arc(0, 0, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#c084fc';
                ctx.fill();
                ctx.strokeStyle = '#7e22ce';
                ctx.lineWidth = 4;
                ctx.stroke();
                ctx.fillStyle = '#fdf2f8';
                for(let i=0; i<4; i++) { ctx.rotate(Math.PI/2); ctx.fillRect(this.radius*0.5, -3, this.radius*0.4, 6); }
                ctx.restore();
            }
        }

        class Drone {
            constructor(x, y, ownerId) {
                this.x = x;
                this.y = y;
                this.ownerId = ownerId;
                this.radius = 10;
                this.active = true;
                this.damageCooldown = 0;
                this.speed = 3.5;
            }

            update(ballsArray) {
                if (!this.active) return;
                let nearestEnemy = null;
                let minDist = Infinity;
                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId && !b.isClone) {
                        let d = Math.hypot(this.x - b.x, this.y - b.y);
                        if (d < minDist) { minDist = d; nearestEnemy = b; }
                    }
                });

                if (nearestEnemy) {
                    let dx = nearestEnemy.x - this.x;
                    let dy = nearestEnemy.y - this.y;
                    let dist = Math.hypot(dx, dy);
                    this.x += (dx / dist) * getEffectiveSpeed(this.speed);
                    this.y += (dy / dist) * getEffectiveSpeed(this.speed);
                    
                    const targetDist = nearestEnemy.radius + this.radius + 0.05;
                    const angle = Math.atan2(this.y - nearestEnemy.y, this.x - nearestEnemy.x);
                    const nextAngle = angle + 0.04 * getEffectiveSpeed(1); 
                    
                    const moveStep = getEffectiveSpeed(this.speed);
                    const newDist = dist > targetDist ? Math.max(targetDist, dist - moveStep) : Math.min(targetDist, dist + moveStep);
                    
                    this.x = nearestEnemy.x + Math.cos(nextAngle) * newDist;
                    this.y = nearestEnemy.y + Math.sin(nextAngle) * newDist;

                    if (dist > targetDist + 5) {
                        this.x += (dx / dist) * getEffectiveSpeed(this.speed);
                        this.y += (dy / dist) * getEffectiveSpeed(this.speed);
                    } else {
                        const angle = Math.atan2(this.y - nearestEnemy.y, this.x - nearestEnemy.x);
                        const nextAngle = angle + 0.05 * getEffectiveSpeed(1);
                        const moveStep = getEffectiveSpeed(this.speed);
                        const newDist = dist > targetDist ? Math.max(targetDist, dist - moveStep) : Math.min(targetDist, dist + moveStep);
                        this.x = nearestEnemy.x + Math.cos(nextAngle) * newDist;
                        this.y = nearestEnemy.y + Math.sin(nextAngle) * newDist;
                    }

                    if (dist < targetDist + 10) {
                        this.damageCooldown--;
                        if (this.damageCooldown <= 0) {
                            nearestEnemy.takeDamage(1);
                            nearestEnemy.takeDamage(2);
                            this.damageCooldown = 60;
                        }
                    }
                }

                if (this.x - this.radius <= 0 || this.x + this.radius >= canvas.width || 
                    this.y - this.radius <= 0 || this.y + this.radius >= canvas.height) {
                    this.active = false;
                }
            }

            draw(ctx) {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(Date.now() / 80);
                ctx.fillStyle = COLORS[TYPE_DRON];
                ctx.fillRect(-12, -2, 24, 4);
                ctx.fillRect(-2, -12, 4, 24);
                ctx.beginPath();
                ctx.arc(0, 0, 5, 0, Math.PI * 2);
                ctx.fillStyle = BORDER_COLORS[TYPE_DRON];
                ctx.fill();
                ctx.restore();
            }
        }

        class RabbitKick {
            constructor(x, y, vx, vy, ownerId) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.ownerId = ownerId;
                this.radius = 18;
                this.active = true;
                this.life = 120; // 2 segundos ida y vuelta total (60fps)
                this.angle = Math.atan2(vy, vx);
            }
            update(ballsArray) {
                this.life--;
                if (this.life <= 0) { this.active = false; return; }

                if (this.life > 60) {
                    // Fase de Ida (1s total, 0.5s para la mitad de la distancia)
                    this.x += this.vx;
                    this.y += this.vy;
                } else {
                    // Fase de Vuelta (0.5s) buscando al dueño
                    const owner = ballsArray.find(b => b.id === this.ownerId);
                    if (owner) {
                        const dx = owner.x - this.x;
                        const dy = owner.y - this.y;
                        this.angle = Math.atan2(dy, dx);
                        this.x += dx / (this.life + 1);
                        this.y += dy / (this.life + 1);
                    } else { this.active = false; }
                }

                for (let b of ballsArray) {
                    if (b.isAlive && b.id !== this.ownerId) {
                        if (Math.hypot(this.x - b.x, this.y - b.y) < this.radius + b.radius) {
                            b.takeDamage(3);
                            this.active = false;
                            break;
                        }
                    }
                }
            }
            draw(ctx) {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);
                ctx.fillStyle = '#fce7f3';
                ctx.strokeStyle = '#ec4899';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.ellipse(0, 0, 15, 8, 0, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();
                ctx.fillStyle = '#f472b6';
                ctx.beginPath(); ctx.arc(8, 0, 3, 0, Math.PI*2); ctx.fill();
                ctx.restore();
            }
        }

        class Carrot {
            constructor(x, y, vx, vy, ownerId) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.ownerId = ownerId;
                this.radius = 12;
                this.active = true;
                this.life = 600;
            }
            update(ballsArray) {
                this.x += this.vx;
                this.y += this.vy;
                this.life--;
                if (this.life <= 0) this.active = false;
                if (this.x < this.radius || this.x > canvas.width - this.radius) this.vx *= -1;
                if (this.y < this.radius || this.y > canvas.height - this.radius) this.vy *= -1;
                for (let b of ballsArray) {
                    if (b.isAlive) {
                        let d = Math.hypot(this.x - b.x, this.y - b.y);
                        if (d < this.radius + b.radius) {
                            if (b.id === this.ownerId) {
                                b.hp = Math.min(b.maxHp, b.hp + 4);
                                floatingTexts.push(new FloatingText(b.x, b.y - 40, "+4 HP", '#22c55e'));
                            } else { b.takeDamage(7); }
                            this.active = false;
                            break;
                        }
                    }
                }
            }
            draw(ctx) {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(Math.atan2(this.vy, this.vx) + Math.PI/4);
                ctx.fillStyle = '#f97316';
                ctx.beginPath(); ctx.moveTo(15, 0); ctx.lineTo(-10, -8); ctx.lineTo(-10, 8); ctx.closePath(); ctx.fill();
                ctx.fillStyle = '#22c55e'; ctx.fillRect(-14, -4, 6, 8);
                ctx.restore();
            }
        }

        class DarkSoul {
            constructor(x, y, ownerId) {
                this.x = x;
                this.y = y;
                this.ownerId = ownerId;
                this.radius = BALL_RADIUS * 1.4;
                this.life = 240;
                this.active = true;
            }

            update(ballsArray) {
                this.life--;
                if (this.life <= 0) { this.active = false; return; }

                for (const b of ballsArray) {
                    if (b.isAlive && b.id === this.ownerId && b.type === TYPE_MUERTE) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            b.soulsCollected++;
                            floatingTexts.push(new FloatingText(this.x, this.y, "💀 ALMA", '#a855f7'));
                            this.active = false;
                            break;
                        }
                    }
                }
            }

            draw(ctx) {
                const alpha = Math.min(1, this.life / 60);
                ctx.save();
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                const grad = ctx.createRadialGradient(this.x, this.y, 0, this.x, this.y, this.radius);
                grad.addColorStop(0, `rgba(76, 29, 149, ${alpha * 0.8})`);
                grad.addColorStop(1, `rgba(0, 0, 0, 0)`);
                ctx.fillStyle = grad;
                ctx.fill();
                ctx.strokeStyle = `rgba(168, 85, 247, ${alpha})`;
                ctx.lineWidth = 2;
                ctx.setLineDash([5, 5]);
                ctx.stroke();
                ctx.restore();
            }
        }

        class CrystalShard {
            constructor(x, y, vx, vy, ownerId) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.ownerId = ownerId;
                this.radius = 6;
                this.active = true;
                this.life = 210;
                this.friction = 0.96;
                this.isBeingAttracted = false;
                this.attractTarget = null;
            }

            update(ballsArray) {
                this.life--;
                if (this.life <= 0) { this.active = false; return; }

                if (this.isBeingAttracted && this.attractTarget && this.attractTarget.isAlive) {
                    let dx = this.attractTarget.x - this.x;
                    let dy = this.attractTarget.y - this.y;
                    let dist = Math.hypot(dx, dy);
                    if (dist < this.attractTarget.radius + this.radius) {
                        this.attractTarget.hp = Math.min(this.attractTarget.maxHp, this.attractTarget.hp + 0.5);
                        this.active = false;
                        return;
                    }
                    this.x += (dx / dist) * getEffectiveSpeed(10);
                    this.y += (dy / dist) * getEffectiveSpeed(10);
                    return;
                }

                if (this.life > 180) {
                    this.x += this.vx;
                    this.y += this.vy;
                    this.vx *= this.friction;
                    this.vy *= this.friction;

                    if (this.x - this.radius < 0 || this.x + this.radius > canvas.width) this.vx *= -1;
                    if (this.y - this.radius < 0 || this.y + this.radius > canvas.height) this.vy *= -1;
                }

                for (const b of ballsArray) {
                    if (b.isAlive && b.id !== this.ownerId) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            b.takeDamage(2);
                            this.active = false;
                            break;
                        }
                    }
                }
            }

            draw(ctx) {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(226, 232, 240, ${Math.min(1, this.life / 60)})`;
                ctx.fill();
                ctx.strokeStyle = '#94a3b8';
                ctx.lineWidth = 1;
                ctx.stroke();
                ctx.closePath();
            }
        }

        class OrbitalPlanet {
            constructor(owner, orbitRadius, orbitSpeed) {
                this.owner = owner;
                this.orbitRadius = orbitRadius;
                this.orbitSpeed = orbitSpeed;
                this.angle = Math.random() * Math.PI * 2;
                this.radius = 12;
                this.x = 0;
                this.y = 0;
                this.active = true;
                this.color = `hsl(${Math.random() * 360}, 70%, 60%)`; 
            }

            update(ballsArray) {
                if (!this.owner.isAlive) { this.active = false; return; }
                
                this.angle += this.orbitSpeed;
                this.x = this.owner.x + Math.cos(this.angle) * this.orbitRadius;
                this.y = this.owner.y + Math.sin(this.angle) * this.orbitRadius;

                for (const b of ballsArray) {
                    if (b.isAlive && b.id !== this.owner.id) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            b.takeDamage(4);
                            this.active = false;
                            break;
                        }
                    }
                }
            }

            draw(ctx) {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.strokeStyle = 'rgba(0,0,0,0.3)';
                ctx.stroke();
            }
        }

        class WoodenPlank {
            constructor(side, ownerId) {
                this.side = side;
                this.ownerId = ownerId;
                this.life = 540;
                this.maxLife = 540;
                this.thickness = 22;
                this.length = canvas.width * 0.28;

                if (side === 'left') {
                    this.x = 0;
                    this.y = Math.random() * (canvas.height - this.thickness);
                    this.w = this.length;
                    this.h = this.thickness;
                } else if (side === 'right') {
                    this.x = canvas.width - this.length;
                    this.y = Math.random() * (canvas.height - this.thickness);
                    this.w = this.length;
                    this.h = this.thickness;
                } else if (side === 'top') {
                    this.x = Math.random() * (canvas.width - this.thickness);
                    this.y = 0;
                    this.w = this.thickness;
                    this.h = this.length;
                } else if (side === 'bottom') {
                    this.x = Math.random() * (canvas.width - this.thickness);
                    this.y = canvas.height - this.length;
                    this.w = this.thickness;
                    this.h = this.length;
                }
            }

            update(ballsArray) {
                this.life--;
                for (const b of ballsArray) {
                    if (b.isAlive && b.id !== this.ownerId) {
                        let closestX = Math.max(this.x, Math.min(b.x, this.x + this.w));
                        let closestY = Math.max(this.y, Math.min(b.y, this.y + this.h));
                        let distance = Math.hypot(b.x - closestX, b.y - closestY);

                        if (distance < b.radius) {
                            if (b.id === this.ownerId) {
                                this.life = 0;
                                return;
                            } else {
                                b.takeDamage(4);
                                this.life = 0;
                                return; 
                            }
                        }
                    }
                }
            }
            draw(ctx) {
                const alpha = Math.min(1, this.life / 60);
                ctx.save();
                ctx.globalAlpha = alpha;
                ctx.fillStyle = '#78350f';
                ctx.fillRect(this.x, this.y, this.w, this.h);
                ctx.strokeStyle = '#451a03';
                ctx.lineWidth = 2;
                ctx.strokeRect(this.x, this.y, this.w, this.h);
                ctx.restore();
            }
         }
          
        class SpiderThread {
            constructor(x1, y1, x2, y2, ownerId) {
                this.x1 = x1;
                this.y1 = y1;
                this.x2 = x2;
                this.y2 = y2;
                this.ownerId = ownerId;
                this.life = 660;
                this.damageCooldowns = new Map();
            }

            update(ballsArray) {
                this.life--;
                const owner = ballsArray.find(b => b.id === this.ownerId);
                if (owner && owner.isAlive) {
                    this.x2 = owner.x;
                    this.y2 = owner.y;
                }

                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId) {
                        if (this.distToSegment(b.x, b.y, this.x1, this.y1, this.x2, this.y2) < b.radius) {
                            let cd = this.damageCooldowns.get(b.id) || 0;
                            if (cd <= 0) {
                                b.takeDamage(1, balls.find(ob => ob.id === this.ownerId));
                                this.damageCooldowns.set(b.id, 30);
                            }
                        }
                    }
                    if (this.damageCooldowns.has(b.id)) {
                        let cd = this.damageCooldowns.get(b.id);
                        if (cd > 0) this.damageCooldowns.set(b.id, cd - 1);
                    }
                });
            }

            distToSegment(px, py, x1, y1, x2, y2) {
                let l2 = (x2 - x1)**2 + (y2 - y1)**2;
                if (l2 === 0) return Math.hypot(px - x1, py - y1);
                let t = ((px - x1) * (x2 - x1) + (py - y1) * (y2 - y1)) / l2;
                t = Math.max(0, Math.min(1, t));
                return Math.hypot(px - (x1 + t * (x2 - x1)), py - (y1 + t * (y2 - y1)));
            }

            draw(ctx) {
                const alpha = Math.min(1, this.life / 60);
                ctx.save();
                ctx.beginPath();
                ctx.moveTo(this.x1, this.y1);
                ctx.lineTo(this.x2, this.y2);
                ctx.strokeStyle = `rgba(226, 232, 240, ${alpha * 0.6})`;
                ctx.lineWidth = 3;
                ctx.setLineDash([5, 3]);
                ctx.lineDashOffset = (Date.now() / 50) % 8;
                ctx.stroke();
                ctx.restore();
            }
        }

        const DAMAGE_CODE_SNIPPETS = [
            "b.takeDamage(p.damage, owner);",
            "this.hp -= damageToHp;",
            "e.takeDamage(7, this);",
            "if (dist < this.radius + b.radius)"
        ];
        const HEAL_CODE_SNIPPETS = [
            "this.hp = Math.min(this.maxHp, this.hp + 1);",
            "b.hp = Math.min(b.maxHp, b.hp + 3);",
            "this.hp += heal;",
            "new FloatingText(b.x, b.y, '+3 HP', '#22c55e');"
        ];

        class CodeLine {
            constructor(x, y, vx, vy, text, type, ownerId, value) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.text = text;
                this.type = type;
                this.ownerId = ownerId;
                this.value = value;
                this.life = 120;
                this.maxLife = 120;
                this.active = true;
                this.hitIds = new Set();
            }

            update(ballsArray) {
                this.life--;
                if (this.life <= 0) { this.active = false; return; }

                this.x += this.vx;
                this.y += this.vy;

                ballsArray.forEach(b => {
                    if (b.isAlive && !this.hitIds.has(b.id)) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < b.radius + 15) {
                            if (this.type === 'damage' && b.id !== this.ownerId) {
                                let owner = ballsArray.find(ob => ob.id === this.ownerId);
                                b.takeDamage(this.value, owner);
                                this.hitIds.add(b.id);
                                this.active = false;
                            } else if (this.type === 'heal' && b.id === this.ownerId) {
                                b.hp = Math.min(b.maxHp, b.hp + this.value);
                                floatingTexts.push(new FloatingText(b.x, b.y - 40, `+${Math.round(this.value)} HP`, '#22c55e'));
                                this.hitIds.add(b.id);
                                this.active = false;
                            }
                        }
                    }
                });

                if (this.x < 0 || this.x > canvas.width) this.vx *= -1;
                if (this.y < 0 || this.y > canvas.height) this.vy *= -1;
            }

            draw(ctx) {
                const alpha = Math.min(1, this.life / 60);
                ctx.save();
                ctx.globalAlpha = alpha;
                ctx.fillStyle = this.type === 'damage' ? '#ef4444' : '#22c55e';
                ctx.font = 'bold 14px monospace';
                ctx.fillText(this.text, this.x, this.y);
                ctx.restore();
            }
        }

        class DongEffect {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.life = 45;
                this.maxLife = 45;
                this.active = true;
            }

            update() {
                this.life--;
                if (this.life <= 0) this.active = false;
            }

            draw(ctx) {
                const progress = 1 - (this.life / this.maxLife);
                const radius = progress * Math.max(canvas.width, canvas.height);
                const alpha = (this.life / this.maxLife) * 0.5;
                ctx.save();
                ctx.beginPath();
                ctx.arc(this.x, this.y, radius, 0, Math.PI * 2);
                ctx.strokeStyle = `rgba(253, 230, 138, ${alpha})`;
                ctx.lineWidth = 4;
                ctx.stroke();
                ctx.restore();
            }
        }

        class LaserBeamV2 {
            constructor(x1, y1, x2, y2, ownerId) {
                this.x1 = x1;
                this.y1 = y1;
                this.x2 = x2;
                this.y2 = y2;
                this.ownerId = ownerId;
                this.life = 720;
                this.damageCooldowns = new Map();
            }

            update(ballsArray) {
                this.life--;
                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId) {
                        if (this.distToSegment(b.x, b.y, this.x1, this.y1, this.x2, this.y2) < b.radius) {
                            let cd = this.damageCooldowns.get(b.id) || 0;
                            if (cd <= 0) {
                                b.takeDamage(1);
                                this.damageCooldowns.set(b.id, 30);
                            }
                        }
                    }
                    if (this.damageCooldowns.has(b.id)) {
                        let currentCd = this.damageCooldowns.get(b.id);
                        if (currentCd > 0) this.damageCooldowns.set(b.id, currentCd - 1);
                    }
                });
            }

            distToSegment(px, py, x1, y1, x2, y2) {
                let l2 = (x2 - x1)**2 + (y2 - y1)**2;
                if (l2 === 0) return Math.hypot(px - x1, py - y1);
                let t = ((px - x1) * (x2 - x1) + (py - y1) * (y2 - y1)) / l2;
                t = Math.max(0, Math.min(1, t));
                return Math.hypot(px - (x1 + t * (x2 - x1)), py - (y1 + t * (y2 - y1)));
            }

            draw(ctx) {
                const alpha = Math.min(1, this.life / 60);
                ctx.save();
                ctx.beginPath();
                ctx.moveTo(this.x1, this.y1);
                ctx.lineTo(this.x2, this.y2);
                ctx.strokeStyle = `rgba(34, 211, 238, ${alpha * 0.8})`;
                ctx.lineWidth = 5;
                ctx.setLineDash([15, 5]);
                ctx.lineDashOffset = -(Date.now() / 20) % 20;
                ctx.stroke();
                ctx.restore();
            }
        }

        class Cactus {
            constructor(x, y, ownerId) {
                this.x = x;
                this.y = y;
                this.ownerId = ownerId;
                this.radius = 18;
                this.life = 360;
                this.active = true;
            }

            update(ballsArray) {
                this.life--;
                if (this.life <= 0) { this.active = false; return; }

                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            b.takeDamage(5);
                            this.active = false;
                        }
                    }
                });
            }

            draw(ctx) {
                if (!this.active) return;
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.fillStyle = '#2d5a27';
                ctx.beginPath();
                ctx.fillRect(-6, -18, 12, 36);
                ctx.fillRect(-14, -8, 8, 6);
                ctx.fillRect(-14, -14, 6, 12);
                ctx.fillRect(6, 2, 8, 6);
                ctx.fillRect(8, -4, 6, 12);
                ctx.restore();
            }
        }

        class SonicWave {
            constructor(x, y, ownerId) {
                this.x = x;
                this.y = y;
                this.ownerId = ownerId;
                this.maxRadius = BALL_RADIUS * 4;
                this.currentRadius = BALL_RADIUS;
                this.life = 120;
                this.maxLife = 120;
                this.damageCooldown = 0;
            }

            update(ballsArray) {
                this.life--;
                this.currentRadius = BALL_RADIUS + (this.maxRadius - BALL_RADIUS) * (1 - this.life / this.maxLife);

                this.damageCooldown--;
                if (this.damageCooldown <= 0) {
                    ballsArray.forEach(b => {
                        if (b.isAlive && b.id !== this.ownerId) {
                            let dist = Math.hypot(this.x - b.x, this.y - b.y);
                            if (dist < this.currentRadius + b.radius) {
                                b.takeDamage(2);
                            }
                        }
                    });
                    this.damageCooldown = 15;
                }
            }

            draw(ctx) {
                const alpha = (this.life / this.maxLife) * 0.5;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.currentRadius, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(56, 189, 248, ${alpha})`;
                ctx.fill();
                ctx.strokeStyle = `rgba(224, 242, 254, ${alpha * 2})`;
                ctx.lineWidth = 2;
                ctx.stroke();
                ctx.closePath();
            }
        }

        class Apple {
            constructor(x, y, ownerId) {
                this.x = x;
                this.y = y;
                this.ownerId = ownerId;
                this.radius = 12;
                this.life = 600;
                this.active = true;
            }

            update(ballsArray) {
                this.life--;
                if (!this.active) return;

                ballsArray.forEach(b => {
                    if (b.isAlive) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            if (b.id === this.ownerId) {
                                b.hp = Math.min(b.maxHp, b.hp + 3);
                                b.speedBoostTimer = 60;
                                floatingTexts.push(new FloatingText(b.x, b.y - 40, "+3 HP & VELOCIDAD!", '#22c55e'));
                            } else {
                                b.takeDamage(4);
                                floatingTexts.push(new FloatingText(b.x, b.y - 40, "¡MANZANA PODRIDA!", '#ef4444'));
                            }
                            this.active = false;
                        }
                    }
                });
            }

            draw(ctx) {
                if (!this.active) return;
                ctx.save();
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#ef4444';
                ctx.fill();
                ctx.fillStyle = '#22c55e';
                ctx.fillRect(this.x - 2, this.y - this.radius - 2, 4, 6);
                ctx.restore();
            }
        }

        class Boomerang {
            constructor(x, y, owner, target) {
                this.x = x;
                this.y = y;
                this.owner = owner;
                this.target = target;
                this.radius = 12;
                this.life = 240;
                this.active = true;
                this.speed = 4;
                this.speed = 8;
                this.damageCooldowns = new Map();

                this.destX = target.x;
                this.destY = target.y;
                this.state = 0;
                this.waitTimer = 12;
                this.destReturnX = 0;
                this.destReturnY = 0;
            }

            update(ballsArray) {
                this.life--;
                if (this.life <= 0 || !this.target.isAlive) { this.active = false; return; }

                if (this.state === 0) {
                    let dx = this.destX - this.x;
                    let dy = this.destY - this.y;
                    let dist = Math.hypot(dx, dy);

                    if (dist < 10) {
                        this.state = 1;
                    } else {
                        this.x += (dx / dist) * getEffectiveSpeed(this.speed);
                        this.y += (dy / dist) * getEffectiveSpeed(this.speed);
                    }
                } 
                else if (this.state === 1) {
                    this.waitTimer--;
                    if (this.waitTimer <= 0) {
                        this.state = 2;
                        this.damageCooldowns.clear();
                        this.destReturnX = this.owner.x;
                        this.destReturnY = this.owner.y;
                    }
                } 
                else if (this.state === 2) {
                    let dx = this.destReturnX - this.x;
                    let dy = this.destReturnY - this.y;
                    let dist = Math.hypot(dx, dy);

                    if (dist > 10) {
                        this.x += (dx / dist) * getEffectiveSpeed(this.speed);
                        this.y += (dy / dist) * getEffectiveSpeed(this.speed);
                    } else {
                        this.active = false;
                    }
                }

                ballsArray.forEach(b => {
                    if (b.isAlive) {
                        let d = Math.hypot(this.x - b.x, this.y - b.y);
                        if (d < this.radius + b.radius) {
                            if (b.id !== this.owner.id) {
                                let cooldown = this.damageCooldowns.get(b.id) || 0;
                                if (cooldown <= 0) {
                                    b.takeDamage(4);
                                    this.damageCooldowns.set(b.id, 12);
                                } else {
                                    this.damageCooldowns.set(b.id, cooldown - 1);
                                }
                            } else if (this.state === 2) {
                                this.active = false;
                            }
                        } else {
                            this.damageCooldowns.delete(b.id);
                        }
                    }
                });
            }

            draw(ctx) {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(Date.now() / 80);
                ctx.beginPath();
                ctx.moveTo(-15, -5);
                ctx.lineTo(15, 0);
                ctx.lineTo(-15, 15);
                ctx.strokeStyle = '#94a3b8';
                ctx.lineWidth = 5;
                ctx.lineCap = 'round';
                ctx.stroke();
                ctx.restore();
            }
        }

        class Vine {
            constructor(x, y, ownerId) {
                this.x = x;
                this.y = y;
                this.ownerId = ownerId;
                this.radius = BALL_RADIUS * 0.7;
                this.life = 180;
                this.maxLife = 180;
                this.active = true;
            }

            update(ballsArray) {
                this.life--;
                if (!this.active) return;

                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId && b.grabbedTimer <= 0) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            b.grabbedTimer = 120;
                            b.isGrabbedByVine = true;
                            this.active = false;
                            floatingTexts.push(new FloatingText(b.x, b.y - 40, "¡ENREDADO!", '#166534'));
                        }
                    }
                });
            }

            draw(ctx) {
                if (!this.active) return;
                const alpha = (this.life / this.maxLife) * 0.8;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(34, 197, 94, ${alpha})`;
                ctx.fill();
                ctx.strokeStyle = `rgba(22, 101, 52, ${alpha})`;
                ctx.lineWidth = 3;
                ctx.stroke();
                ctx.closePath();
            }
        }

        class ToxicZone {
            constructor(x, y, side, ownerId) {
                this.x = x;
                this.y = y;
                this.side = side;
                this.ownerId = ownerId;
                this.radius = BALL_RADIUS * 0.6; 
                this.life = 1200;
                this.maxLife = 1200;
                this.damageCooldown = 30;
            }

            update(ballsArray) {
                this.life--;
                if (this.life <= 0) return;

                this.damageCooldown--;
                if (this.damageCooldown <= 0) {
                    ballsArray.forEach(b => {
                        if (b.isAlive && b.id !== this.ownerId) {
                            let dist = Math.hypot(this.x - b.x, this.y - b.y);
                            if (dist < this.radius + b.radius) {
                                b.takeDamage(2);
                            }
                        }
                    });
                    this.damageCooldown = 30;
                }
            }

            draw(ctx) {
                const alpha = this.life / this.maxLife * 0.6;
                ctx.beginPath();
                if (this.side === 'left') {
                    ctx.moveTo(0, this.y - this.radius);
                    ctx.lineTo(0, this.y + this.radius);
                    ctx.lineTo(this.radius * 1.5, this.y);
                } else if (this.side === 'right') {
                    ctx.moveTo(canvas.width, this.y - this.radius);
                    ctx.lineTo(canvas.width, this.y + this.radius);
                    ctx.lineTo(canvas.width - this.radius * 1.5, this.y);
                } else if (this.side === 'top') {
                    ctx.moveTo(this.x - this.radius, 0);
                    ctx.lineTo(this.x + this.radius, 0);
                    ctx.lineTo(this.x, this.radius * 1.5);
                } else if (this.side === 'bottom') {
                    ctx.moveTo(this.x - this.radius, canvas.height);
                    ctx.lineTo(this.x + this.radius, canvas.height);
                    ctx.lineTo(this.x, canvas.height - this.radius * 1.5);
                }
                ctx.closePath();
                ctx.fillStyle = `rgba(144, 238, 144, ${alpha})`;
                ctx.fill();
                ctx.closePath();
            }
        }

        class ExplosionEffect {
            constructor(x, y, radius, duration, ownerId) {
                this.x = x;
                this.y = y;
                this.radius = radius;
                this.life = duration;
                this.maxLife = duration;
                this.ownerId = ownerId;
                this.hitBallIds = new Set();
            }

            update(ballsArray) {
                this.life--;
                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId && !this.hitBallIds.has(b.id)) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            b.takeDamage(10);
                            this.hitBallIds.add(b.id);
                        }
                    }
                });
            }

            draw(ctx) {
                const alpha = this.life / this.maxLife * 0.8;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(255, 165, 0, ${alpha})`;
                ctx.fill();
                ctx.closePath();
            }
        }

        class RadioactiveTrail {
            constructor(x, y, ownerId) {
                this.x = x;
                this.y = y;
                this.ownerId = ownerId;
                this.radius = BALL_RADIUS / 2;
                this.life = 240;
                this.maxLife = 240;
                this.damage = 3;
                this.hitBallIds = new Set();
            }

            update(ballsArray) {
                this.life--;
                if (this.life <= 0) return;

                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId && !this.hitBallIds.has(b.id)) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            b.takeDamage(this.damage);
                            this.hitBallIds.add(b.id);
                        }
                    }
                });
            }

            draw(ctx) {
                const alpha = (this.life / this.maxLife) * 0.7;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(57, 255, 20, ${alpha})`;
                ctx.fill();
                ctx.closePath();
            }
        }

        class WaterPuddle {
            constructor(x, y, ownerId) {
                this.x = x;
                this.y = y;
                this.ownerId = ownerId;
                this.radius = BALL_RADIUS * 1.6;
                this.life = 240;
                this.maxLife = 240;
                this.hitBallIds = new Set();
            }

            update(ballsArray) {
                this.life--;
                if (this.life <= 0) return;

                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId && !this.hitBallIds.has(b.id)) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            b.takeDamage(2);
                            b.startSlipping();
                            this.hitBallIds.add(b.id);
                        }
                    }
                });
            }

            draw(ctx) {
                const alpha = (this.life / this.maxLife) * 0.5;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(14, 165, 233, ${alpha})`;
                ctx.fill();
                ctx.strokeStyle = `rgba(255, 255, 255, ${alpha * 0.5})`;
                ctx.stroke();
                ctx.closePath();
            }
        }

        class Fireball {
            constructor(x, y, vx, vy, ownerId) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.ownerId = ownerId;
                this.radius = 12;
                this.life = 240;
                this.active = true;
            }

            update(ballsArray) {
                this.x += this.vx;
                this.y += this.vy;
                this.life--;
                if (this.life <= 0) this.active = false;

                if (this.x - this.radius < 0) { this.x = this.radius; this.vx *= -1; }
                if (this.x + this.radius > canvas.width) { this.x = canvas.width - this.radius; this.vx *= -1; }
                if (this.y - this.radius < 0) { this.y = this.radius; this.vy *= -1; }
                if (this.y + this.radius > canvas.height) { this.y = canvas.height - this.radius; this.vy *= -1; }

                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId) {
                        let dist = Math.hypot(this.x - b.x, this.y - b.y);
                        if (dist < this.radius + b.radius) {
                            b.applyBurn();
                            this.active = false;
                        }
                    }
                });
            }

            draw(ctx) {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = '#ff4500';
                ctx.fill();
                ctx.closePath();
            }
        }

        class ClosedSpace {
            constructor(x, y, ownerId) {
                this.x = x;
                this.y = y;
                this.ownerId = ownerId;
                this.radius = BALL_RADIUS * 2.2;
                this.life = 150;
                this.maxLife = 150;
                this.damage = 2;
            }

            update(ballsArray) {
                this.life--;
                if (this.life <= 0) return;

                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.ownerId) {
                        let dx = b.x - this.x;
                        let dy = b.y - this.y;
                        let dist = Math.hypot(dx, dy);

                        if (dist + b.radius >= this.radius && dist < this.radius) {
                            let nx = dx / dist;
                            let ny = dy / dist;
                            let dot = b.vx * nx + b.vy * ny;

                            if (dot > 0) {
                                b.vx -= 2 * dot * nx;
                                b.vy -= 2 * dot * ny;
                                b.takeDamage(this.damage);
                            }
                        }
                    }
                });
            }

            draw(ctx) {
                const alpha = (this.life / this.maxLife) * 0.3;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(147, 51, 234, ${alpha})`;
                ctx.fill();
                ctx.strokeStyle = `rgba(147, 51, 234, ${alpha * 2})`;
                ctx.lineWidth = 6;
                ctx.setLineDash([10, 5]);
                ctx.stroke();
                ctx.setLineDash([]);
                ctx.closePath();
            }
        }

        class Ball {
            constructor(id, type, controller, startX, startY, name) {
                this.id = id;
                this.name = name || `J${id}`;
                this.type = type;
                this.controller = controller; 
                this.x = startX;
                this.y = startY;
                this.vx = 0;
                this.vy = 0;
                this.radius = BALL_RADIUS;
                this.hp = 100;
                this.maxHp = 100;
                if (type === TYPE_DOBLE) {
                    this.hp = 50;
                    this.maxHp = 50;
                }
                this.isAlive = true;
                this.color = COLORS[type];
                this.borderColor = BORDER_COLORS[type];
                
                // Propiedades de Clonación
                this.isClone = false;
                this.cloneOwner = null;
                this.activeClone = null;
                
                // Habilidades
                this.cooldown = 0;
                this.isDashing = false;
                this.dashTimer = 0;
                this.invulnTimer = 0; 
                this.launchedTimer = 0; // Para el lanzamiento de carnívora

                // Propiedades de Pelota Plantadora
                this.grabbedTimer = 0;
                this.isGrabbedByVine = false;
                this.isTrappedByJelly = false;
                
                // Propiedades de Pelota de Agua
                this.slipTimer = 0; 
                this.waterCooldown = 0;

                // Habilidades de Pelota de Fuego
                this.burnTimer = 0;
                this.burnDamageCooldown = 0;
                this.poisonTimer = 0;
                this.poisonDamageCooldown = 0;

                // Estados de Debuff (Nuevos para Pelota Color)
                this.silenceTimer = 0;
                this.slowTimer = 0;
                this.weaknessTimer = 0;

                // Habilidades de Pelota Carnívora
                this.carnivoreTarget = null;
                this.carnivoreGrabTimer = 0;
                this.carnivoreCooldown = 0;
                this.carnivoreDamageCooldown = 0;

                // Habilidades de Pelota Vampiro
                this.vampireAttachedTo = null; 
                this.vampireAttachTimer = 0; 
                this.vampireDrainCooldown = 0; 
                this.vampireAbilityCooldown = 0; 

                // Habilidades de Pelota Explosiva
                this.explosionCooldown = 240; 

                // Habilidades de Pelota Bomba
                this.bombActive = false;
                this.bombTimer = 0;
                this.bombCarrier = null;
                this.bombCooldown = 0;
                this.bombTransferCooldown = 0; 

                // Habilidades de Pelota Tóxica
                this.lastHitWall = false; 

                // Habilidades de Pelota Láser
                this.laserTarget = null; 
                this.laserDamageCooldown = 0; 
                this.laserRange = BALL_RADIUS * 5.75; 

                // Habilidades de Pelota Espadachina
                this.swordAngle = Math.random() * Math.PI * 2; 
                this.swordOrbitRadius = this.radius + 15; 
                this.swordLength = this.radius * 2.0; 
                this.swordRotationSpeed = 0.05; 
                this.swordHitCooldown = 0; 

                // Habilidades de Pelota Radioactiva
                this.trailCreationCooldown = 15; 
                this.lastTrailDrop = 0;

                // Habilidades de Pelota Magnética
                this.magnetPhase = 'attract'; 
                this.magnetTimer = 180; 
                this.magnetRange = BALL_RADIUS * 4.5;

                // Habilidades de Pelota Fantasma
                this.isGhostMode = false;
                this.ghostTimer = 0;
                this.ghostDamageCooldown = 0;

                // Habilidades de Pelota Eléctrica
                this.charges = 0;
                this.chargeTimer = 120; 

                // Habilidades de Pelota de Hielo
                this.iceAuraActive = false;
                this.iceAuraTimer = 0;
                this.iceCooldown = 300; // 5 segundos
                this.isFrozen = false;
                this.timeInIceAura = 0;

                // Propiedades de Pelota Aleatoria
                this.randomDamage = 5;
                this.randomDamageTimer = 60;

                // Propiedades de Pelota Manzana
                this.speedBoostTimer = 0;

                // Propiedades de Pelota Portal
                this.portalCooldown = 90; // 1.5 segundos
                this.teleportDamageRadius = BALL_RADIUS * 2.04; // +20%
                this.portalNextX = null;
                this.portalNextY = null;

                // Propiedades de Pelota Bicho
                this.bichoAttackTimer = 0;
                this.bichoAttackAngle = 0;
                this.bichoHitIds = new Set();
                this.bichoAttackCount = 0;
                this.desertTimer = 0;
                this.laserAnchor = null; 
                this.shieldHp = 20;
                this.shieldMaxHp = 20;
                this.shieldCooldown = 0;
                this.pendingCounterDamage = 0; 

                this.luck = 0.05; 
                this.coinTossCooldown = 60; 
                this.jackpotHealTimer = 0;
                this.jackpotDamageTimer = 0;
                this.jackpotCount = 0; 

                this.slideTimer = 0; 
                this.slideMultiplier = 1;
                this.slideDamage = 0; 
                this.metalShotsLeft = 0;
                this.metalShotTimer = 0;
                this.machineGunCount = 0;
                this.machineGunTimer = 0;
                this.crystalAttractTimer = 600; 

                this.soulsCollected = 0;
                this.soulSpawnTimer = 180; 
                this.deathAttackTimer = 1800; 

                this.isFurious = false;

                // Propiedades de Pelota Topo
                this.topoPhase = 0; 
                this.topoTargetX = 0;
                this.topoTargetY = 0;
                this.topoHitIds = new Set();
                this.topoCooldown = 360; 

                this.programDamageTimer = 240; 
                this.programHealTimer = 360; 

                // Propiedades Pelota Error
                this.errorTimer = 0;
                this.currentErrorType = null;
                this.errorCooldown = 0;

                // Propiedades Pelota Gravitacional
                this.gravityTimer = 0;
                this.gravityCooldown = 600; 
                this.gravityPushTimer = 0; 
                this.gravityHitWallCooldown = 0; 

                // Propiedades Pelota Divisora
                this.divisoraTimer = 0;
                this.divisoraCooldown = 600; 
                this.divisoraQuadrant = -1; 
                this.divisoraDamageTimer = 0;
                this.lastHitByTeam = {};
                this.deathTimer = 0;
                this.sandiaCooldown = 300;
                
                // Propiedades Pelota Golem
                if (type === TYPE_GOLEM) this.radius = BALL_RADIUS * 1.2;
                this.golemCooldown = 210; // 3.5 segundos
                this.golemAttackTimer = 0;
                this.golemHitIds = new Set();

                // Propiedades Pelota de Tiempo
                this.timeStopTimer = 0;
                this.timeStopCooldown = 0;

                // Propiedades Pelota Rayo
                this.rayoTimer = 210; // 3.5 segundos
                this.rayoStrikeStage = 0; 
                this.rayoStrikes = [];
                this.rayoStrikeCount = 0;

                // Propiedades Pelota de Piedra
                this.stoneTimer = 420; 
                this.stoneDuration = 0;
                this.isPetrified = false;
                this.stoneAreaDamageTimer = 0;

                // Propiedades de Pelota Dimensional
                this.dimensionalCooldown = 420; // 7 seconds * 60 fps
                this.dimensionalCooldown = 210; // 3.5 seconds * 60 fps
                this.teleportEffectActive = false;
                this.teleportEffectTimer = 0; // Counts down the 5 seconds

                // Propiedades Pelota Conejo
                this.rabbitKickTimer = 120; // 2s
                this.rabbitCarrotTimer = 720; // 12s
                this.rabbitPendingKick = 0;
                this.rabbitKickAngle = 0;

                // Propiedades Pelota Numérica
                this.numericTimer = 0;
                this.numericCooldown = 0;
                this.numericGrid = [];

                // Propiedades Pelota Proyección
                this.projectionTimer = 0; // Fase de congelamiento (90 frames = 1.5s)
                this.projectionCooldown = 0; // Cooldown (330 frames = 5.5s)
                this.projectionPath = [];
                this.projectionMoving = false;
                this.nuclearTimer = 2700; // 45 segundos * 60 fps
                this.discoCooldown = 330; // 5.5 segundos
                this.wolfTimer = 0;
                this.wolfCooldown = 630; // 10.5 segundos para la primera transformación
                this.discoCooldown = 360; // 6 segundos
                this.spaceTimer = 0;
                this.spaceCooldown = 420; // 7 segundos
                this.meteoriteSpawnTimer = 0;
                this.lavaCooldown = 180; // 3 segundos
                this.spiritualTimer = 4000; // 90 segundos
                this.spiritualAttackTimer = 102; // 1.7 segundos
                this.spiritualAttackTimer = 114; // 1.9 segundos (1.7 + 0.2)
                this.spiritualAttackTimer = 131; // ~2.18 segundos (1.9s + 15%)
                this.hpDecayCounter = 0;
                this.spiritualStunTimer = 0;

                // Propiedades Pelota Cuestionadora
                this.questionTimer = 0;
                this.questionCooldown = 0;
                this.questionSide = 'left'; 

                // Propiedades Pelota Absorbedora
                this.absorbedDamage = 0;
                this.absorbTimer = 6600; // 1 minuto y 50 segundos (110s * 60fps)

                // Propiedades Pelota Sol
                this.solStartupTimer = 300; // 5 segundos iniciales
                this.solActive = false;
                this.solLevel = 0; // 0 a 5
                this.solUpgradeTimer = 300; // Cada 5 segundos sube de nivel
                this.solAttackTimer = 0;
                this.solRange = BALL_RADIUS * 1.5;
                this.solDamage = 1;
                this.solCadence = 180; // 3 segundos iniciales (en frames)
                this.solUltActive = false;
                this.solUltTimer = 0;

                // Propiedades Pelota Color
                this.colorCycleTimer = 0;
                this.activeColors = [];
                this.colorHealTimer = 0;
                this.colorHealCount = 0;

                // Propiedades Pelota Compra y Venta
                this.money = 0;
                this.moneyTimer = 60; // 1 segundo
                this.leverUses = 0;
                this.extraDamage = 0;
                this.pistolTimer = 0;
                this.pistolCooldown = 0;
                if (type === TYPE_COMPRA_VENTA) {
                    this.money = 0;
                }
            }

          update(ballsArray) {
        if (!this.isAlive) return;

        if (this.deathTimer > 0) {
            this.deathTimer--;
            this.vx = 0;
            this.vy = 0;
            if (this.deathTimer === 0) {
                this.isAlive = false;
                if (!this.isClone) ranking.push(this);
                explosionEffects.push(new ExplosionEffect(this.x, this.y, this.radius * 2, 30, this.id));
                floatingTexts.push(new FloatingText(this.x, this.y, "¡BOOM!", '#ef4444'));
            }
            return;
        }

        if (this.type === TYPE_ERROR && this.grabbedTimer <= 0) {
            if (!this.currentErrorType && this.errorCooldown <= 0) {
                const possible = ALL_BALL_TYPES.filter(t => t.value !== 'random_pick' && t.value !== TYPE_ERROR);
                this.currentErrorType = possible[Math.floor(Math.random() * possible.length)].value;
                this.errorTimer = 300;
                floatingTexts.push(new FloatingText(this.x, this.y - 70, `GLITCH: ${this.currentErrorType.toUpperCase()}`, '#d946ef'));

                this.cooldown = 0;
                const timersToReset = ['explosionCooldown', 'bombCooldown', 'shieldCooldown', 'vampireAbilityCooldown', 'iceCooldown', 'waterCooldown', 'carnivoreCooldown', 'portalCooldown', 'topoCooldown', 'programDamageTimer', 'programHealTimer', 'desertTimer', 'bichoAttackTimer', 'crystalAttractTimer', 'deathAttackTimer', 'coinTossCooldown', 'wolfTimer', 'wolfCooldown'];
                timersToReset.forEach(t => { if (this[t] !== undefined) this[t] = 0; });
                
                this.bombActive = false; this.isGhostMode = false; this.iceAuraActive = false; this.topoPhase = 0; this.isDashing = false;
            }
            
            if (this.currentErrorType) {
                this.errorTimer--;
                if (this.errorTimer <= 0) {
                    this.currentErrorType = null;
                    this.errorCooldown = 240; 
                    floatingTexts.push(new FloatingText(this.x, this.y - 70, "REBOOT", '#9ca3af'));
                }
            }
        }
        
        if (this.errorCooldown > 0) this.errorCooldown--;

        const skillType = (this.silenceTimer > 0) ? TYPE_NORMAL : ((this.type === TYPE_ERROR && this.currentErrorType) ? this.currentErrorType : this.type);
        
                // Lógica de Congelamiento Total
                const isThisTimeStopper = (skillType === TYPE_TIEMPO && this.timeStopTimer > 0) || 
                                          (skillType === TYPE_PROYECCION && this.projectionTimer > 0);

                const globalTimeStopped = ballsArray.some(ob => {
                    const obSkill = (ob.type === TYPE_ERROR && ob.currentErrorType) ? ob.currentErrorType : ob.type;
                    return (obSkill === TYPE_TIEMPO && ob.isAlive && ob.timeStopTimer > 0) ||
                           (obSkill === TYPE_PROYECCION && ob.isAlive && ob.projectionTimer > 0);
                });

                if (globalTimeStopped && !isThisTimeStopper) return;

                // Lógica Pelota Color Cycle
                if (skillType === TYPE_COLOR && this.isAlive && this.grabbedTimer <= 0) {
                    this.colorCycleTimer--;
                    if (this.colorCycleTimer <= 0) {
                        this.colorCycleTimer = 360; // 6 segundos
                        const palette = ['rojo', 'verde', 'amarillo', 'morado', 'azul', 'negro'];
                        this.activeColors = [
                            palette[Math.floor(Math.random() * palette.length)],
                            palette[Math.floor(Math.random() * palette.length)]
                        ];
                        
                        // Aplicar efectos instantáneos
                        this.activeColors.forEach(c => {
                            if (c === 'amarillo') {
                                this.shieldHp += 4;
                                floatingTexts.push(new FloatingText(this.x, this.y - 60, "+4 ESCUDO", '#fde047'));
                            }
                            if (c === 'verde') {
                                this.colorHealTimer = 180; // 3 segundos
                                this.colorHealCount = 0;
                            }
                            if (c === 'negro') {
                                let ang = Math.random() * Math.PI * 2;
                                blackBalls.push(new BlackBall(this.x, this.y, Math.cos(ang) * 5, Math.sin(ang) * 5, this.id));
                                floatingTexts.push(new FloatingText(this.x, this.y - 60, "BOLA NEGRA", '#000000'));
                            }
                        });
                        showMessage(`COLOR: ${this.activeColors[0].toUpperCase()} + ${this.activeColors[1].toUpperCase()}`);
                    }

                    // Lógica Curación Verde
                    if (this.colorHealTimer > 0) {
                        this.colorHealTimer--;
                        this.colorHealCount++;
                        if (this.colorHealCount >= 60) {
                            this.hp = Math.min(this.maxHp, this.hp + 1);
                            floatingTexts.push(new FloatingText(this.x, this.y - 40, "+1 HP", '#22c55e'));
                            this.colorHealCount = 0;
                        }
                    }
                }

                // Actualizar Timers de Debuff
                if (this.silenceTimer > 0) {
                    this.silenceTimer--;
                    if (this.silenceTimer === 0) floatingTexts.push(new FloatingText(this.x, this.y - 70, "Habilidades OK", '#a855f7'));
                }
                if (this.slowTimer > 0) {
                    this.slowTimer--;
                    if (this.slowTimer === 0) floatingTexts.push(new FloatingText(this.x, this.y - 70, "Velocidad OK", '#60a5fa'));
                }
                if (this.weaknessTimer > 0) {
                    this.weaknessTimer--;
                    if (this.weaknessTimer === 0) floatingTexts.push(new FloatingText(this.x, this.y - 70, "Daño OK", '#60a5fa'));
                }

                // Override visual si está silenciado
                if (this.silenceTimer > 0) { this.borderColor = '#4b5563'; }

                if (this.speedBoostTimer > 0) this.speedBoostTimer--;

                if (skillType === TYPE_SOL && this.isAlive && this.grabbedTimer <= 0) {
                    if (this.solStartupTimer > 0) {
                        this.solStartupTimer--;
                        if (this.solStartupTimer <= 0) {
                            this.solActive = true;
                            this.solAttackTimer = this.solCadence;
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡SOL NACIENTE!", '#facc15'));
                        }
                    } else if (this.solUltActive) {
                        this.solUltTimer--;
                        if (this.solUltTimer % 30 === 0) { // Cada 0.5 segundos
                            ballsArray.forEach(b => {
                                if (b.isAlive && b.id !== this.id) {
                                    b.takeDamage(1, this, false, false, true);
                                }
                            });
                        }
                        if (this.solUltTimer <= 0) {
                            this.solUltActive = false;
                            this.solLevel = 0;
                            this.solDamage = 1;
                            this.solCadence = 180;
                            this.solRange = BALL_RADIUS * 1.5;
                            this.solUpgradeTimer = 300;
                            this.solActive = true;
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "SOL REINICIADO", '#9ca3af'));
                        }
                    } else if (this.solActive) {
                        // Lógica de escalado
                        this.solUpgradeTimer--;
                        if (this.solUpgradeTimer <= 0) {
                            this.solLevel++;
                            if (this.solLevel <= 5) {
                                this.solDamage += 1;
                                this.solCadence -= 30; // -0.5 segundos (3, 2.5, 2, 1.5, 1)
                                this.solRange += 25;
                                this.solUpgradeTimer = 300;
                                floatingTexts.push(new FloatingText(this.x, this.y - 60, `CALOR NIVEL ${this.solLevel}`, '#fbbf24'));
                            } else {
                                // Sexta vez: Ultimate
                                this.solUltActive = true;
                                this.solUltTimer = 180; // 3 segundos
                                this.solActive = false;
                                floatingTexts.push(new FloatingText(this.x, this.y - 70, "¡SUPERNOVA!", '#ef4444'));
                            }
                        }

                        // Lógica de ataque de aura
                        this.solAttackTimer--;
                        if (this.solAttackTimer <= 0) {
                            let hit = false;
                            ballsArray.forEach(b => {
                                if (b.isAlive && b.id !== this.id) {
                                    if (Math.hypot(this.x - b.x, this.y - b.y) < this.solRange + b.radius) {
                                        b.takeDamage(this.solDamage, this);
                                        hit = true;
                                    }
                                }
                            });
                            if (hit) {
                                // Pequeño efecto visual al golpear
                                explosionEffects.push(new ExplosionEffect(this.x, this.y, this.solRange, 10, this.id));
                            }
                            this.solAttackTimer = this.solCadence;
                        }
                    }
                }

                if (skillType === TYPE_ABSORVEDORA && this.isAlive && this.grabbedTimer <= 0) {
                    if (this.absorbTimer > 0) {
                        this.absorbTimer--;
                        if (this.absorbTimer <= 0) {
                            const enemies = ballsArray.filter(b => b.isAlive && b.id !== this.id);
                            enemies.forEach(e => {
                                e.takeDamage(this.absorbedDamage, this, false, false, true);
                                floatingTexts.push(new FloatingText(e.x, e.y - 60, `¡LIBERACIÓN! -${Math.ceil(this.absorbedDamage)}`, '#10b981'));
                            });
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡ENERGÍA LIBERADA!", '#10b981'));
                            this.absorbedDamage = 0;
                            this.absorbTimer = 6600; // Reinicia el ciclo
                        }
                    }
                }

                if (skillType === TYPE_CUESTIONADORA && this.isAlive && this.grabbedTimer <= 0) {
                    if (this.questionTimer > 0) {
                        this.questionTimer--;
                        if (this.questionTimer <= 0) {
                            const midX = canvas.width / 2;
                            ballsArray.forEach(b => {
                                if (b.isAlive && b.id !== this.id) {
                                    const isLeft = b.x < midX;
                                    const onSiSide = (this.questionSide === 'left' && isLeft) || (this.questionSide === 'right' && !isLeft);
                                    if (onSiSide) {
                                        b.takeDamage(8, this);
                                        floatingTexts.push(new FloatingText(b.x, b.y - 40, "¡SÍ!", '#ef4444'));
                                    } else {
                                        floatingTexts.push(new FloatingText(b.x, b.y - 40, "SALVADO", '#22c55e'));
                                    }
                                }
                            });
                            this.questionCooldown = 420; // 7 segundos
                        }
                    } else if (this.questionCooldown > 0) {
                        this.questionCooldown--;
                    } else {
                        this.questionTimer = 180; // 3 segundos
                        this.questionSide = Math.random() < 0.5 ? 'left' : 'right';
                    }
                }

                if (skillType === TYPE_TIEMPO && this.grabbedTimer <= 0) {
                    if (this.timeStopTimer > 0) {
                        this.timeStopTimer--;
                        if (this.timeStopTimer <= 0) {
                            this.cooldown = 120; 
                        }
                    } else if (this.cooldown <= 0) {
                        this.timeStopTimer = 120; 
                        floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡TIEMPO DETENIDO!", '#6366f1'));
                    }
                }

                if (skillType === TYPE_DIVISORA && this.grabbedTimer <= 0) {
                    if (this.divisoraTimer > 0) {
                        this.divisoraTimer--;
                        
                        this.divisoraDamageTimer--;
                        if (this.divisoraDamageTimer <= 0) {
                            this.divisoraDamageTimer = 30; 
                            
                            const midX = canvas.width / 2;
                            const midY = canvas.height / 2;
                            
                            ballsArray.forEach(b => {
                                if (b.isAlive && b.id !== this.id) {
                                    let inQuadrant = false;
                                    if (this.divisoraQuadrant === 0) inQuadrant = b.x < midX && b.y < midY;
                                    else if (this.divisoraQuadrant === 1) inQuadrant = b.x >= midX && b.y < midY;
                                    else if (this.divisoraQuadrant === 2) inQuadrant = b.x < midX && b.y >= midY;
                                    else if (this.divisoraQuadrant === 3) inQuadrant = b.x >= midX && b.y >= midY;
                                    
                                    if (inQuadrant) b.takeDamage(5, this);
                                }
                            });
                        }
                    } else if (this.divisoraCooldown > 0) {
                        this.divisoraCooldown--;
                    } else {
                        this.divisoraTimer = 360; 
                        this.divisoraCooldown = 600; 
                        this.divisoraQuadrant = Math.floor(Math.random() * 4);
                        this.divisoraDamageTimer = 0;
                        floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡DIVISIÓN!", '#ef4444'));
                    }
                }

                if (skillType === TYPE_RAYO && this.grabbedTimer <= 0) {
                    this.rayoTimer--;
                    if (this.rayoTimer <= 0) {
                        if (this.rayoStrikeStage === 0) {
                            this.rayoStrikeStage = 1;
                            this.rayoTimer = 60; // 1 segundo de espera
                            this.rayoStrikeCount++;
                            this.rayoStrikes = [];
                            const count = (this.rayoStrikeCount % 3 === 0) ? 3 : 1;
                            for (let i = 0; i < count; i++) {
                                this.rayoStrikes.push({ x: Math.random() * canvas.width, y: Math.random() * canvas.height });
                            }
                            this.rayoStrikes.forEach(s => {
                                floatingTexts.push(new FloatingText(s.x, s.y - 60, count === 3 ? "¡SOBRECARGA!" : "¡CARGANDO!", '#fde047'));
                            });
                        } else {
                            const isTriple = (this.rayoStrikeCount % 3 === 0);
                            const strikeRadius = isTriple ? this.radius * 1.4 : this.radius * 2;
                            const dmg = isTriple ? 10 : 8;

                            this.rayoStrikes.forEach(s => {
                                ballsArray.forEach(b => {
                                    if (b.isAlive && b.id !== this.id) {
                                        let dist = Math.hypot(s.x - b.x, s.y - b.y);
                                        if (dist < strikeRadius + b.radius) {
                                            b.takeDamage(dmg, this);
                                        }
                                    }
                                });
                                floatingTexts.push(new FloatingText(s.x, s.y - 60, "¡RAYO!", '#fde047'));
                            });
                            this.rayoStrikeStage = 0;
                            this.rayoTimer = 210; 
                        }
                    }
                }

                if (skillType === TYPE_PIEDRA && this.grabbedTimer <= 0) {
                    if (this.isPetrified) {
                        this.stoneDuration--;
                        
                        // Daño en área (150% del tamaño) cada 0.3 segundos (18 frames)
                        this.stoneAreaDamageTimer--;
                        if (this.stoneAreaDamageTimer <= 0) {
                            this.stoneAreaDamageTimer = 18;
                            const areaRadius = this.radius * 1.5;
                            ballsArray.forEach(b => {
                                if (b.isAlive && b.id !== this.id) {
                                    let dist = Math.hypot(this.x - b.x, this.y - b.y);
                                    if (dist < areaRadius + b.radius) {
                                        b.takeDamage(5, this);
                                    }
                                }
                            });
                        }

                        this.vx = 0;
                        this.vy = 0;
                        this.invulnTimer = 2; 
                        this.color = '#9ca3af'; 
                        this.borderColor = '#4b5563';
                        if (this.stoneDuration <= 0) {
                            this.isPetrified = false;
                            this.stoneTimer = 420;
                            this.color = COLORS[TYPE_PIEDRA];
                            this.borderColor = BORDER_COLORS[TYPE_PIEDRA];
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡NORMAL!", '#a8a29e'));
                        }
                    } else {
                        this.stoneTimer--;
                        if (this.stoneTimer <= 0) {
                            this.isPetrified = true;
                            this.stoneDuration = 300; 
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡PETRIFICADO!", '#9ca3af'));
                        }
                    }
                }

                // Dimensional Ball specific logic
                if (skillType === TYPE_DIMENSIONAL && this.grabbedTimer <= 0) {
                    if (this.teleportEffectActive) {
                        this.teleportEffectTimer--;
                        if (this.teleportEffectTimer <= 0) {
                            this.teleportEffectActive = false;
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "REALIDAD ESTABLE", '#9ca3af'));
                        }
                    } else {
                        if (this.dimensionalCooldown > 0) {
                            this.dimensionalCooldown--;
                        } else {
                            this.teleportEffectActive = true;
                            this.teleportEffectTimer = 300; // 5 seconds * 60 fps
                            this.dimensionalCooldown = 420; // Reset main cooldown for 7 seconds
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡DESGARRO DE REALIDAD!", '#d946ef'));
                        }
                    }
                }

                if (skillType === TYPE_CONEJO && this.grabbedTimer <= 0) {
                    this.rabbitKickTimer--;
                    if (this.rabbitKickTimer <= 0) {
                        this.rabbitKickAngle = Math.random() * Math.PI * 2;
                        const vx = Math.cos(this.rabbitKickAngle) * getEffectiveSpeed(9);
                        const vy = Math.sin(this.rabbitKickAngle) * getEffectiveSpeed(9);
                        rabbitKicks.push(new RabbitKick(this.x, this.y, vx, vy, this.id));
                        this.rabbitPendingKick = 12; // Retraso entre patadas
                        this.rabbitKickTimer = 120;
                    }
                    if (this.rabbitPendingKick > 0) {
                        this.rabbitPendingKick--;
                        if (this.rabbitPendingKick === 0) {
                            const vx = Math.cos(this.rabbitKickAngle) * getEffectiveSpeed(9);
                            const vy = Math.sin(this.rabbitKickAngle) * getEffectiveSpeed(9);
                            // Un poco separada (detrás)
                            const ox = -Math.cos(this.rabbitKickAngle) * 20;
                            const oy = -Math.sin(this.rabbitKickAngle) * 20;
                            rabbitKicks.push(new RabbitKick(this.x + ox, this.y + oy, vx, vy, this.id));
                        }
                    }
                    this.rabbitCarrotTimer--;
                    if (this.rabbitCarrotTimer <= 0) {
                        let angle = Math.random() * Math.PI * 2;
                        carrots.push(new Carrot(this.x, this.y, Math.cos(angle) * getEffectiveSpeed(6), Math.sin(angle) * getEffectiveSpeed(6), this.id));
                        this.rabbitCarrotTimer = 720;
                        floatingTexts.push(new FloatingText(this.x, this.y - 50, "¡ZANAHORIA!", '#f97316'));
                    }
                }

                if (skillType === TYPE_NUMERICA && this.grabbedTimer <= 0) {
                    if (this.numericTimer > 0) {
                        this.numericTimer--;
                    } else if (this.numericCooldown > 0) {
                        this.numericCooldown--;
                    } else {
                        // Activa por 7 segundos (420 frames), cooldown de 8 segundos (480 frames)
                        this.numericTimer = 420;
                        this.numericCooldown = 480;
                        this.numericGrid = [1, 2, 3, 4, 5, 6, 7, 8, 9].sort(() => Math.random() - 0.5);
                        floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡CÁLCULO MORTAL!", '#22d3ee'));
                    }
                }

                if (skillType === TYPE_PROYECCION && this.grabbedTimer <= 0) {
                    if (this.projectionTimer > 0) {
                        this.projectionTimer--;
                        if (this.projectionTimer <= 0) {
                            this.projectionMoving = true;
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡PROYECCIÓN!", '#818cf8'));
                        }
                        return; // Se queda congelada en su lugar durante el trazado
                    } else if (this.projectionMoving) {
                        if (this.projectionPath.length > 0) {
                            const target = this.projectionPath[0];
                            const dx = target.x - this.x;
                            const dy = target.y - this.y;
                            const dist = Math.hypot(dx, dy);
                            const speed = getEffectiveSpeed(20);

                            if (dist < speed) {
                                this.x = target.x;
                                this.y = target.y;
                                this.projectionPath.shift();
                            } else {
                                this.vx = (dx / dist) * speed;
                                this.vy = (dy / dist) * speed;
                                this.x += this.vx;
                                this.y += this.vy;
                            }
                        } else {
                            this.projectionMoving = false;
                        this.projectionCooldown = 330; // 5.5 segundos
                            // Retoma movimiento normal
                            let angle = Math.random() * Math.PI * 2;
                            this.vx = Math.cos(angle) * getEffectiveSpeed(BASE_SPEED);
                            this.vy = Math.sin(angle) * getEffectiveSpeed(BASE_SPEED);
                        }
                    } else if (this.projectionCooldown > 0) {
                        this.projectionCooldown--;
                    } else {
                        this.projectionTimer = 90; // 1.5s
                        this.projectionPath = [];
                        for (let i = 0; i < 5; i++) {
                            this.projectionPath.push({
                                x: Math.random() * (canvas.width - this.radius * 2) + this.radius,
                                y: Math.random() * (canvas.height - this.radius * 2) + this.radius
                            });
                        }
                    }
                }

                if (skillType === TYPE_NUCLEAR && this.isAlive && this.grabbedTimer <= 0) {
                    if (this.nuclearTimer > 0) {
                        this.nuclearTimer--;
                        if (this.nuclearTimer === 0) {
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡MELTDOWN NUCLEAR!", '#4ade80'));
                        }
                    } else {
                        this.nuclearTimer--; // Seguimos decrementando para usar el %60
                        if (Math.abs(this.nuclearTimer) % 60 === 0) {
                            ballsArray.forEach(b => {
                                if (b.isAlive && b.id !== this.id && !b.isClone) {
                                    b.takeDamage(2, this, false, false, true);
                                }
                            });
                        }
                    }
                }

                if (skillType === TYPE_HOMBRELOBO && this.grabbedTimer <= 0) {
                    if (this.wolfTimer > 0) {
                        this.wolfTimer--;
                        if (this.wolfTimer <= 0) {
                            this.wolfCooldown = 630;
                            this.color = COLORS[TYPE_HOMBRELOBO];
                            this.borderColor = BORDER_COLORS[TYPE_HOMBRELOBO];
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "HUMANO", '#94a3b8'));
                        }
                    } else if (this.wolfCooldown > 0) {
                        this.wolfCooldown--;
                    } else {
                        this.wolfTimer = 360; // 6 segundos de transformación
                        this.color = '#78350f'; // Marrón lobuno
                        this.borderColor = '#451a03';
                        floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡LOBO!", '#ef4444'));
                    }
                }

                if (skillType === TYPE_ESPACIO && this.grabbedTimer <= 0) {
                    if (this.spaceTimer > 0) {
                        this.spaceTimer--;
                        this.meteoriteSpawnTimer--;
                        if (this.meteoriteSpawnTimer <= 0) {
                            const side = Math.floor(Math.random() * 4);
                            let x, y, vx, vy;
                            const speed = getEffectiveSpeed(5);
                            if (side === 0) { // Arriba
                                x = Math.random() * canvas.width; y = -50;
                                vx = (Math.random() - 0.5) * 4; vy = speed;
                            } else if (side === 1) { // Derecha
                                x = canvas.width + 50; y = Math.random() * canvas.height;
                                vx = -speed; vy = (Math.random() - 0.5) * 4;
                            } else if (side === 2) { // Abajo
                                x = Math.random() * canvas.width; y = canvas.height + 50;
                                vx = (Math.random() - 0.5) * 4; vy = -speed;
                            } else { // Izquierda
                                x = -50; y = Math.random() * canvas.height;
                                vx = speed; vy = (Math.random() - 0.5) * 4;
                            }
                            meteorites.push(new Meteorite(x, y, vx, vy, this.id));
                            this.meteoriteSpawnTimer = 15 + Math.random() * 45; // 0.25 a 1 seg
                        }
                        if (this.spaceTimer <= 0) {
                            this.spaceCooldown = 420;
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "REGRESO A TIERRA", '#94a3b8'));
                        }
                    } else if (this.spaceCooldown > 0) {
                        this.spaceCooldown--;
                    } else {
                        this.spaceTimer = 780; // 13 segundos (10s + 3s extra)
                        this.meteoriteSpawnTimer = 0;
                        floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡EL ESPACIO!", '#818cf8'));
                    }
                }

                if (skillType === TYPE_LAVA && this.grabbedTimer <= 0) {
                    this.lavaCooldown--;
                    if (this.lavaCooldown <= 0) {
                        for (let i = 0; i < 2; i++) {
                            const sides = ['left', 'right', 'top', 'bottom'];
                            const side = sides[Math.floor(Math.random() * sides.length)];
                            let vx, vy;
                            if (side === 'left') { vx = 0; vy = Math.random() * canvas.height; }
                            else if (side === 'right') { vx = canvas.width; vy = Math.random() * canvas.height; }
                            else if (side === 'top') { vx = Math.random() * canvas.width; vy = 0; }
                            else { vx = Math.random() * canvas.width; vy = canvas.height; }

                            lavaVolcanoes.push(new Volcano(vx, vy, side, this.id));
                        }
                        this.lavaCooldown = 180;
                    }
                }

                if (skillType === TYPE_ESPIRITUAL && this.isAlive && this.grabbedTimer <= 0) {
                    if (this.spiritualStunTimer > 0) this.spiritualStunTimer--;

                    if (this.spiritualTimer > 0) {
                        this.spiritualTimer--;
                        this.invulnTimer = 2; // Inmortalidad mientras tenga tiempo
                        
                        if (this.spiritualStunTimer <= 0) {
                            this.spiritualAttackTimer--;
                            if (this.spiritualAttackTimer <= 0) {
                                this.spawnSpiritualSouls(ballsArray);
                                this.spiritualAttackTimer = 102; // 1.7s
                                this.spiritualAttackTimer = 114; // 1.9s
                                this.spiritualAttackTimer = 131; // 2.18s
                            }
                        }
                    } else {
                        // Modo Desesperación
                        if (this.spiritualStunTimer <= 0) {
                            this.spiritualAttackTimer--;
                            if (this.spiritualAttackTimer <= 0) {
                                this.spawnSpiritualSouls(ballsArray);
                                this.spiritualAttackTimer = 24; // 0.4s
                                this.spiritualAttackTimer = 57; // 100% más rápido que el estado normal (114 / 2)
                                this.spiritualAttackTimer = 66; // 100% más rápido que el estado normal (131 / 2)
                            }
                        }
                        this.hpDecayCounter++;
                        if (this.hpDecayCounter >= 60) {
                            this.takeDamage(10, null, false, false, true);
                            this.hpDecayCounter = 0;
                        }
                    }
                }

                if (this.grabbedTimer > 0) {
                    this.grabbedTimer--;
                    this.vx = 0;
                    this.vy = 0;

                    if (this.type === TYPE_VAMPIRO) this.vampireAttachedTo = null;
                    if (this.type === TYPE_CARNIVORA) this.carnivoreTarget = null;
                    if (this.type === TYPE_LASER) this.laserTarget = null;
                    if (this.type === TYPE_RAPIDA) this.isDashing = false;
                    if (this.type === TYPE_FANTASMA) { this.isGhostMode = false; this.radius = BALL_RADIUS; }

                    if (this.grabbedTimer % 60 === 0) this.takeDamage(this.isTrappedByJelly ? 2 : 1);

                    if (this.grabbedTimer === 0) {
                        let angle = Math.random() * Math.PI * 2;
                        this.vx = Math.cos(angle) * getEffectiveSpeed(BASE_SPEED);
                        this.vy = Math.sin(angle) * getEffectiveSpeed(BASE_SPEED);
                        this.isGrabbedByVine = false;
                        this.isTrappedByJelly = false;
                        this.isFrozen = false;
                    }
                }

                if (skillType === TYPE_GRAVITACIONAL && this.grabbedTimer <= 0) {
                    if (this.gravityTimer > 0) {
                        this.gravityTimer--;
                        this.invulnTimer = 2; 
                        
                        let enemies = ballsArray.filter(b => b.id !== this.id && b.isAlive);
                        enemies.forEach(e => {
                            if (e.gravityHitWallCooldown > 0) return;

                            const dLeft = e.x;
                            const dRight = canvas.width - e.x;
                            const dTop = e.y;
                            const dBottom = canvas.height - e.y;
                            const minDist = Math.min(dLeft, dRight, dTop, dBottom);
                            
                            const force = getEffectiveSpeed(0.5);
                            if (minDist === dLeft) e.vx -= force;
                            else if (minDist === dRight) e.vx += force;
                            else if (minDist === dTop) e.vy -= force;
                            else if (minDist === dBottom) e.vy += force;
                            
                            e.gravityPushTimer = 2; 
                        });
                    } else if (this.gravityCooldown > 0) {
                        this.gravityCooldown--;
                    } else {
                        this.gravityTimer = 300; 
                        this.gravityCooldown = 600; 
                        floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡GRAVEDAD!", '#38bdf8'));
                    }
                }
                if (this.gravityPushTimer > 0) this.gravityPushTimer--;
                if (this.gravityHitWallCooldown > 0) this.gravityHitWallCooldown--;

                if (skillType === TYPE_ESCUDERA && this.pendingCounterDamage > 0 && this.grabbedTimer <= 0) {
                    let nearestEnemy = null;
                    let minDist = Infinity;

                if ((skillType === TYPE_ESCUDERA || skillType === TYPE_ARMADA) && this.shieldHp <= 0) {
                    this.shieldCooldown--;
                    if (this.shieldCooldown <= 0) {
                        this.shieldHp = this.shieldMaxHp;
                        floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡ESCUDO RECARGADO!", '#cbd5e1'));
                    }
                }
                    ballsArray.forEach(b => {
                        if (b.isAlive && b.id !== this.id && !b.isClone) {
                            let dist = Math.hypot(this.x - b.x, this.y - b.y);
                            if (dist < minDist) {
                                minDist = dist;
                                nearestEnemy = b;
                            }
                        }
                    });
                    if (nearestEnemy) {
                        nearestEnemy.takeDamage(this.pendingCounterDamage, this);
                        floatingTexts.push(new FloatingText(nearestEnemy.x, nearestEnemy.y - 60, `¡VENGANZA! -${Math.ceil(this.pendingCounterDamage)}`, '#ef4444'));
                    }
                    this.pendingCounterDamage = 0;
                }

                if (skillType === TYPE_APOSTADORA && this.isAlive && this.grabbedTimer <= 0) {
                    if (this.jackpotHealTimer <= 0 && this.jackpotDamageTimer <= 0) {
                        this.coinTossCooldown--;
                        if (this.coinTossCooldown <= 0) {
                            this.coinTossCooldown = 60; 
                            if (Math.random() < this.luck) {
                                this.jackpotHealTimer = 240; 
                                this.jackpotDamageTimer = 360; 
                                this.jackpotCount++;
                                if (this.jackpotCount % 2 === 0) {
                                    this.luck += 0.01; 
                                }
                                floatingTexts.push(new FloatingText(this.x, this.y - 50, "¡JACKPOT! 💰", '#fbbf24'));
                            } else {
                                floatingTexts.push(new FloatingText(this.x, this.y - 45, "🪙", '#9ca3af'));
                                if (this.hp > 20) {
                                    this.takeDamage(1);
                                }
                            }
                        }
                    }
                    
                    if (this.jackpotHealTimer > 0) {
                        this.jackpotHealTimer--;
                        if (this.jackpotHealTimer % 16 === 0) this.hp = Math.min(this.maxHp, this.hp + 1);
                    }
                    if (this.jackpotDamageTimer > 0) this.jackpotDamageTimer--;
                }

                if (this.burnTimer > 0) {
                    this.burnTimer--;
                    if (this.burnDamageCooldown > 0) this.burnDamageCooldown--;
                    if (this.burnDamageCooldown <= 0) {
                        this.takeDamage(3);
                        this.burnDamageCooldown = 60; 
                    }
                }

                if (this.poisonTimer > 0) {
                    this.poisonTimer--;
                    if (this.poisonDamageCooldown > 0) this.poisonDamageCooldown--;
                    if (this.poisonDamageCooldown <= 0) {
                        this.takeDamage(1);
                        this.poisonDamageCooldown = 60; 
                    }
                }

                if (skillType === TYPE_CLONADORA && !this.isClone && this.isAlive && this.grabbedTimer <= 0) {
                    if (this.cooldown <= 0) {
                        const clone = new Ball(this.id, TYPE_CLONADORA, 'bot', this.x, this.y, this.name);
                        clone.isClone = true;
                        clone.radius = this.radius * 0.7; 
                        clone.hp = 5;
                        clone.maxHp = 10;
                        clone.cloneOwner = this;
                        clone.vx = -this.vx || 2;
                        clone.vy = -this.vy || 2;
                        this.activeClone = clone;
                        ballsArray.push(clone);
                
                        this.cooldown = 480; 
                    }
                }

                if (this.cooldown > 0) this.cooldown--;
                if (this.carnivoreCooldown > 0) this.carnivoreCooldown--;
                if (this.waterCooldown > 0) this.waterCooldown--;

                if (skillType === TYPE_PORTAL) {
                    this.portalCooldown--;

                    if (this.portalCooldown === 30) {
                        this.portalNextX = this.radius + Math.random() * (canvas.width - this.radius * 2);
                        this.portalNextY = this.radius + Math.random() * (canvas.height - this.radius * 2);
                    }

                    if (this.portalCooldown <= 0) {
                        const oldX = this.x;
                        const oldY = this.y;
                        this.x = this.portalNextX || (this.radius + Math.random() * (canvas.width - this.radius * 2));
                        this.y = this.portalNextY || (this.radius + Math.random() * (canvas.height - this.radius * 2));
                        this.portalNextX = null;
                        this.portalNextY = null;
                        
                        this.invulnTimer = 30; 
                        ballsArray.forEach(b => {
                            if (b.isAlive && b.id !== this.id) {
                                let dist = Math.hypot(this.x - b.x, this.y - b.y);
                                if (dist < this.teleportDamageRadius + b.radius) {
                                    b.takeDamage(4);
                                    floatingTexts.push(new FloatingText(b.x, b.y - 20, "-4", '#8b5cf6')); 
                                }
                            }
                        });
                        floatingTexts.push(new FloatingText(oldX, oldY - 40, "¡ZAP!", '#8b5cf6')); 
                        floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡BAM!", '#8b5cf6')); 
                        this.portalCooldown = 90; 
                    }
                }
                if (this.invulnTimer > 0) this.invulnTimer--;

                if (skillType === TYPE_VAMPIRO && this.vampireAttachedTo) {
                    this.x = this.vampireAttachedTo.x;
                    this.y = this.vampireAttachedTo.y;
                    this.vx = this.vampireAttachedTo.vx; 
                    this.vy = this.vampireAttachedTo.vy;
                } else if (this.grabbedTimer > 0) {
                    // No se mueve si está atrapado
                } else { 
                    this.x += this.vx;
                    this.y += this.vy;
                }

                // Efecto de movimiento aleatorio del espacio
                const activeSpaceOwner = ballsArray.find(ob => {
                    const obSkill = (ob.type === TYPE_ERROR && ob.currentErrorType) ? ob.currentErrorType : ob.type;
                    return obSkill === TYPE_ESPACIO && ob.isAlive && ob.spaceTimer > 0;
                });
                if (activeSpaceOwner && activeSpaceOwner.id !== this.id && this.grabbedTimer <= 0 && !this.isPetrified) {
                    if (Math.random() < 0.05) {
                        let angle = Math.random() * Math.PI * 2;
                        let speed = Math.hypot(this.vx, this.vy) || getEffectiveSpeed(BASE_SPEED);
                        this.vx = Math.cos(angle) * speed;
                        this.vy = Math.sin(angle) * speed;
                    }
                }

                // Rebote en paredes
                // Check for wall collisions and apply Dimensional Ball effects
                let hitWall = false;
                if (this.x - this.radius < 0) { this.x = this.radius; this.vx *= -1; hitWall = true; }
                if (this.x + this.radius > canvas.width) { this.x = canvas.width - this.radius; this.vx *= -1; hitWall = true; }
                if (this.y - this.radius < 0) { this.y = this.radius; this.vy *= -1; hitWall = true; }
                if (this.y + this.radius > canvas.height) { this.y = canvas.height - this.radius; this.vy *= -1; hitWall = true; }

                if (hitWall) {
                    const activeDimensionalBall = balls.find(b =>
                        (b.type === TYPE_DIMENSIONAL || (b.type === TYPE_ERROR && b.currentErrorType === TYPE_DIMENSIONAL)) &&
                        b.isAlive && b.teleportEffectActive
                    );

                    if (activeDimensionalBall) {
                        if (this.id === activeDimensionalBall.id) { // This ball is the Dimensional Ball itself
                            this.hp = Math.min(this.maxHp, this.hp + 1);
                            floatingTexts.push(new FloatingText(this.x, this.y - 40, "+1 HP (Muro)", '#3b82f6'));
                        } else { // This ball is an enemy
                            this.takeDamage(3); // Damage for hitting wall
                            floatingTexts.push(new FloatingText(this.x, this.y - 40, "-3 (Choque Muro)", '#ef4444'));
                        }
                        this.x = this.radius + Math.random() * (canvas.width - this.radius * 2);
                        this.y = this.radius + Math.random() * (canvas.height - this.radius * 2);
                        let angle = Math.random() * Math.PI * 2;
                        this.vx = Math.cos(angle) * getEffectiveSpeed(BASE_SPEED);
                        this.vy = Math.sin(angle) * getEffectiveSpeed(BASE_SPEED);
                        floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡TELETRANSPORTE!", '#d946ef'));
                    }
                }

                // Rebote en paredes
                if (hitWall && this.gravityPushTimer > 0) {
                    this.takeDamage(4);
                    this.gravityHitWallCooldown = 18; 
                }

                if (hitWall && skillType === TYPE_ARANA) {
                    let wallX = (this.x <= this.radius) ? 0 : (this.x >= canvas.width - this.radius ? canvas.width : this.x);
                    let wallY = (this.y <= this.radius) ? 0 : (this.y >= canvas.height - this.radius ? canvas.height : this.y);
                    
                    spiderThreads.push(new SpiderThread(wallX, wallY, this.x, this.y, this.id));
                }

                if (hitWall && skillType === TYPE_DESLIZANTE && this.cooldown <= 0) {
                    this.slideTimer = 60; 
                    this.slideMultiplier *= 1.7; 
                    this.slideDamage += 1; 
                    if (this.slideMultiplier > 10) this.slideMultiplier = 10;
                    
                    floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡DESLIZAMIENTO!", '#2dd4bf'));
                }

                if (hitWall && this.slipTimer > 0) {
                    this.takeDamage(4);
                    this.slipTimer = 0; 
                }

                // Disminuir timers de efectos de movimiento
                if (this.slipTimer > 0) this.slipTimer--;
                if (this.slideTimer > 0) {
                    this.slideTimer--;
                    if (this.slideTimer <= 0) {
                        this.slideMultiplier = 1;
                        this.slideDamage = 0;
                    }
                }
                if (this.launchedTimer > 0) this.launchedTimer--;

                // ======= GESTOR ROBUSTO DE VELOCIDAD =======
                const isCarnivoreVictim = ballsArray.some(other => 
                    other.isAlive && 
                    ((other.type === TYPE_CARNIVORA) || (other.type === TYPE_ERROR && other.currentErrorType === TYPE_CARNIVORA)) && 
                    other.carnivoreTarget === this
                );

                if (this.grabbedTimer > 0 || isCarnivoreVictim || this.isPetrified) {
                    // Si está atrapada (enredadera, hielo, gelatina o mandíbulas), no forzar velocidad aquí
                } else if (this.vampireAttachedTo) {
                    // Sigue al objetivo (ya se maneja en el bloque superior)
                } else if (skillType === TYPE_TOPO && this.topoPhase > 0) {
                    // El topo bajo tierra tiene su propia lógica de movimiento en su habilidad
                } else if (this.isDashing || this.launchedTimer > 0) {
                    // El dash y los lanzamientos mantienen su impulso temporalmente
                } else {
                    // Determinar la velocidad objetivo correcta
                    let targetSpeed = BASE_SPEED;

                    if (this.slipTimer > 0) {
                        targetSpeed = BASE_SPEED * 1.8;
                    } else if (this.slideTimer > 0) {
                        targetSpeed = BASE_SPEED * this.slideMultiplier;
                    } else {
                        if (skillType === TYPE_GOLEM) targetSpeed *= 0.5;
                        if (this.slowTimer > 0) targetSpeed *= 0.5; // Efecto Pelota Color Azul
                        if (this.speedBoostTimer > 0) targetSpeed = BASE_SPEED * 1.5;
                        if (skillType === TYPE_FURIA && this.isFurious) targetSpeed *= 2;
                        if (skillType === TYPE_HOMBRELOBO && this.wolfTimer > 0) targetSpeed *= 1.5;
                        
                        // Ralentización por Tormenta de Arena
                        const isUnderDesertStorm = ballsArray.some(other => 
                            other.type === TYPE_DESERTICA && other.isAlive && 
                            other.id !== this.id && other.desertTimer > 0
                        );
                        if (isUnderDesertStorm) targetSpeed *= 0.5;
                    }

                    let effectiveTarget = getEffectiveSpeed(targetSpeed);
                    let speed = Math.hypot(this.vx, this.vy);

                    if (speed === 0) {
                        // Prevención de bug de paralización completa
                        let angle = Math.random() * Math.PI * 2;
                        this.vx = Math.cos(angle) * effectiveTarget;
                        this.vy = Math.sin(angle) * effectiveTarget;
                    } else if (Math.abs(speed - effectiveTarget) > 0.01) {
                        // Forzar SIEMPRE la velocidad base o la modificada según los efectos activos
                        this.vx = (this.vx / speed) * effectiveTarget;
                        this.vy = (this.vy / speed) * effectiveTarget;
                    }
                }
                // ===========================================

                let enemies = ballsArray.filter(b => b.id !== this.id && b.isAlive);
                
                if (enemies.length > 0 && this.grabbedTimer <= 0) {
                    let nearestEnemy = null;
                    let minDist = Infinity;
                    enemies.forEach(e => {
                        let dist = Math.hypot(this.x - e.x, this.y - e.y);
                        if (dist < minDist) { minDist = dist; nearestEnemy = e; }
                    });

                    if (skillType === TYPE_AGUA) {
                        if (this.waterCooldown <= 0) {
                            waterPuddles.push(new WaterPuddle(this.x, this.y, this.id));
                            this.waterCooldown = 210; 
                        }
                    }

                    if (skillType === TYPE_FUEGO) {
                        if (this.cooldown <= 0) {
                            let angle = Math.random() * Math.PI * 2;
                            fireballs.push(new Fireball(
                                this.x, 
                                this.y,
                                Math.cos(angle) * getEffectiveSpeed(6),
                                Math.sin(angle) * getEffectiveSpeed(6),
                                this.id
                            ));
                            this.cooldown = 180; 
                        }
                    }

                    if (skillType === TYPE_PLANTADORA) {
                        if (this.cooldown <= 0) {
                            for (let i = 0; i < 3; i++) {
                                let rx = 50 + Math.random() * (canvas.width - 100);
                                let ry = 50 + Math.random() * (canvas.height - 100);
                                vines.push(new Vine(rx, ry, this.id));
                            }
                            this.cooldown = 300; 
                        }
                    }
                    
                    if (skillType === TYPE_SONORA) {
                        if (this.cooldown <= 0) {
                            sonicWaves.push(new SonicWave(this.x, this.y, this.id));
                            this.cooldown = 240; 
                        }
                    }

                    if (skillType === TYPE_CARNIVORA) {
                        if (this.carnivoreTarget) {
                            this.carnivoreGrabTimer--;
                            
                            let angle = Math.atan2(this.y - this.carnivoreTarget.y, this.x - this.carnivoreTarget.x);
                            this.carnivoreTarget.x = this.x - Math.cos(angle) * (this.radius + this.carnivoreTarget.radius) * 0.4;
                            this.carnivoreTarget.y = this.y - Math.sin(angle) * (this.radius + this.carnivoreTarget.radius) * 0.4;
                            this.carnivoreTarget.vx = 0;
                            this.carnivoreTarget.vy = 0;

                            if (this.carnivoreDamageCooldown > 0) this.carnivoreDamageCooldown--;
                            if (this.carnivoreDamageCooldown <= 0) {
                                this.carnivoreTarget.takeDamage(1);
                                this.carnivoreDamageCooldown = 30;
                            }

                            if (this.carnivoreGrabTimer <= 0 || !this.carnivoreTarget.isAlive) {
                                if (this.carnivoreTarget.isAlive) {
                                    let launchAngle = Math.atan2(this.carnivoreTarget.y - this.y, this.carnivoreTarget.x - this.x);
                                    this.carnivoreTarget.vx = Math.cos(launchAngle) * getEffectiveSpeed(12);
                                    this.carnivoreTarget.vy = Math.sin(launchAngle) * getEffectiveSpeed(12);
                                    this.carnivoreTarget.launchedTimer = 30; // Mantiene el impulso temporalmente tras ser soltada
                                    floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡LANZADO!", '#be185d'));
                                }
                                this.carnivoreTarget = null;
                                this.carnivoreCooldown = 210; 
                                this.invulnTimer = 20; 
                            }
                        } else if (this.carnivoreCooldown <= 0 && this.invulnTimer <= 0) {
                            let huntRange = this.radius * 2.2; 
                            let target = enemies.find(e => Math.hypot(this.x - e.x, this.y - e.y) < huntRange);
                            if (target) {
                                this.carnivoreTarget = target;
                                this.carnivoreGrabTimer = 120; 
                                this.carnivoreDamageCooldown = 0;
                                floatingTexts.push(new FloatingText(this.x, this.y - 50, "¡ATRAPADO!", '#be185d'));
                            }
                        }
                    }

                    if (skillType === TYPE_ARMADA) {
                        if (minDist < 250 && this.cooldown <= 0) {
                            let angle = Math.atan2(nearestEnemy.y - this.y, nearestEnemy.x - this.x);
                            projectiles.push(new Projectile(
                                this.x + Math.cos(angle)*(this.radius + 5), 
                                this.y + Math.sin(angle)*(this.radius + 5),
                                Math.cos(angle) * getEffectiveSpeed(PROYECTILE_SPEED * 2),
                                Math.sin(angle) * getEffectiveSpeed(PROYECTILE_SPEED * 2),
                                this.id,
                                3 
                            ));
                            this.cooldown = 150; 
                        }
                    } 
                    else if (skillType === TYPE_CANON) {
                        if (this.cooldown <= 0 && nearestEnemy) {
                            const cannonRadius = BALL_RADIUS * 1.4; 
                            let angle = Math.atan2(nearestEnemy.y - this.y, nearestEnemy.x - this.x);
                            projectiles.push(new Projectile(
                                this.x + Math.cos(angle) * (this.radius + cannonRadius + 5),
                                this.y + Math.sin(angle) * (this.radius + cannonRadius + 5),
                                Math.cos(angle) * getEffectiveSpeed(PROYECTILE_SPEED * 0.5), 
                                Math.sin(angle) * getEffectiveSpeed(PROYECTILE_SPEED * 0.5),
                                this.id,
                                10, 
                                cannonRadius,
                                '#4b5563' 
                            ));
                            this.cooldown = 480; 
                        }
                    }
                    else if (skillType === TYPE_AMETRALLADORA) {
                        if (this.cooldown <= 0 && minDist < 350) {
                            this.machineGunTimer--;
                            if (this.machineGunTimer <= 0) {
                                this.machineGunCount++;
                                const isBigShot = this.machineGunCount >= 15;
                                const dmg = 1;
                                
                                let angle = Math.atan2(nearestEnemy.y - this.y, nearestEnemy.x - this.x);
                                projectiles.push(new Projectile(
                                    this.x + Math.cos(angle)*(this.radius + 5), 
                                    this.y + Math.sin(angle)*(this.radius + 5),
                                    Math.cos(angle) * getEffectiveSpeed(PROYECTILE_SPEED * 2.6),
                                    Math.sin(angle) * getEffectiveSpeed(PROYECTILE_SPEED * 2.6),
                                    this.id,
                                    dmg
                                ));

                                if (isBigShot) {
                                    this.machineGunCount = 0;
                                    this.cooldown = 840; 
                                    floatingTexts.push(new FloatingText(this.x, this.y - 40, "RECARGANDO...", '#94a3b8'));
                                } else {
                                    this.machineGunTimer = 30; 
                                }
                            }
                        }
                    }
                    else if (skillType === TYPE_RAPIDA) {
                        if (this.isDashing) {
                            this.dashTimer--;
                            if (this.dashTimer <= 0) {
                                this.isDashing = false;
                            }
                        } else if (this.cooldown <= 0 && minDist < 350) {
                            let angle = Math.atan2(nearestEnemy.y - this.y, nearestEnemy.x - this.x);
                            this.vx = Math.cos(angle) * getEffectiveSpeed(DASH_SPEED);
                            this.vy = Math.sin(angle) * getEffectiveSpeed(DASH_SPEED);
                            this.isDashing = true;
                            this.dashTimer = 45; 
                            this.cooldown = 180; 
                            this.cooldown = 60; 
                            this.cooldown = 210; // 3.5 segundos
                            this.invulnTimer = this.dashTimer; 
                        }
                    }
                    else if (skillType === TYPE_VAMPIRO) { 
                        if (this.vampireAbilityCooldown > 0) this.vampireAbilityCooldown--;
                        if (this.vampireAttachedTo) {
                            this.vampireAttachTimer--;
                            if (this.vampireAttachTimer <= 0 || !this.vampireAttachedTo.isAlive) {
                                this.vampireAttachedTo = null;
                                this.vampireAbilityCooldown = 180;
                                this.vampireAbilityCooldown = 60;
                                this.vampireAbilityCooldown = 258; // 4.3s * 60fps
                                let angle = Math.random() * Math.PI * 2;
                                this.vx = Math.cos(angle) * getEffectiveSpeed(BASE_SPEED);
                                this.vy = Math.sin(angle) * getEffectiveSpeed(BASE_SPEED);
                            } else {
                                const angle = Math.atan2(this.y - this.vampireAttachedTo.y, this.x - this.vampireAttachedTo.x) || 0;
                                this.x = this.vampireAttachedTo.x + Math.cos(angle) * (this.radius * 0.3);
                                this.y = this.vampireAttachedTo.y + Math.sin(angle) * (this.radius * 0.2);
                                
                                if (--this.vampireDrainCooldown <= 0) {
                                    const damage = 1.5;
                                    this.vampireAttachedTo.takeDamage(damage);
                                    this.hp = Math.min(this.maxHp, this.hp + 1); 
                                    floatingTexts.push(new FloatingText(this.x, this.y - 40, `+1 HP`, '#22c55e'));
                                    this.vampireDrainCooldown = 30;
                                }
                            }
                        }
                    } else if (skillType === TYPE_EXPLOSIVA) {
                        this.explosionCooldown--;
                        if (this.explosionCooldown <= 0) {
                            floatingTexts.push(new FloatingText(this.x, this.y, "BOOM!", '#ff4500'));
                            
                            explosionEffects.push(new ExplosionEffect(
                                this.x,
                                this.y,
                                this.radius * EXPLOSION_RADIUS_MULTIPLIER,
                                EXPLOSION_EFFECT_DURATION,
                                this.id
                            ));
                            this.explosionCooldown = 180;
                        }
                    } else if (skillType === TYPE_BOMBA) {
                        if (this.bombActive) {
                            this.bombTimer--;
                            if (this.bombTransferCooldown > 0) this.bombTransferCooldown--;

                            if (!this.bombCarrier || !this.bombCarrier.isAlive) this.bombCarrier = this;

                            if (this.bombTransferCooldown <= 0) {
                                let others = ballsArray.filter(b => b.isAlive && b !== this.bombCarrier);
                                if (others.length > 0) {
                                    let minDist = Infinity;
                                    let nearestToCarrier = null;
                                    others.forEach(e => {
                                        let d = Math.hypot(this.bombCarrier.x - e.x, this.bombCarrier.y - e.y);
                                        if (d < minDist) { minDist = d; nearestToCarrier = e; }
                                    });

                                    if (nearestToCarrier && minDist < (this.bombCarrier.radius + nearestToCarrier.radius) * 1.1) {
                                        this.bombCarrier = nearestToCarrier;
                                        this.bombTransferCooldown = 30; 
                                        floatingTexts.push(new FloatingText(this.bombCarrier.x, this.bombCarrier.y - 50, "¡BOMBA!", '#ef4444'));
                                    }
                                }
                            }

                            if (this.bombTimer <= 0) {
                                let damage = (this.bombCarrier === this) ? 5 : 20;
                                this.bombCarrier.takeDamage(damage);
                                explosionEffects.push(new ExplosionEffect(this.bombCarrier.x, this.bombCarrier.y, this.radius * 1.5, 30, this.bombCarrier.id));
                                this.bombActive = false;
                                this.bombCooldown = 120; 
                                this.bombCarrier = null;
                            }
                        } else if (this.bombCooldown > 0) {
                            this.bombCooldown--;
                        } else {
                            this.bombActive = true;
                            this.bombTimer = 600; 
                            this.bombCarrier = this;
                            this.bombTransferCooldown = 30; 
                        }
                    } else if (skillType === TYPE_TOXICA) {
                        let side = null;
                        if (this.x <= this.radius) side = 'left';
                        else if (this.x >= canvas.width - this.radius) side = 'right';
                        else if (this.y <= this.radius) side = 'top';
                        else if (this.y >= canvas.height - this.radius) side = 'bottom';

                        if (side && !this.lastHitWall) {
                            toxicZones.push(new ToxicZone(this.x, this.y, side, this.id));
                        }
                        this.lastHitWall = !!side;
                    } else if (skillType === TYPE_LASER) {
                        if (nearestEnemy && minDist < this.laserRange) {
                            this.laserTarget = nearestEnemy;
                            this.laserDamageCooldown--;
                            if (this.laserDamageCooldown <= 0) {
                                this.laserTarget.takeDamage(1);
                                this.laserDamageCooldown = 30; 
                            }
                        } else { this.laserTarget = null; }
                    } else if (skillType === TYPE_ESPADACHINA) {
                        this.swordAngle += this.swordRotationSpeed;
                        if (this.swordHitCooldown > 0) this.swordHitCooldown--;
                    }
                    else if (skillType === TYPE_RADIOACTIVA) {
                        this.lastTrailDrop--;
                        if (this.lastTrailDrop <= 0) {
                            radioactiveTrails.push(new RadioactiveTrail(this.x, this.y, this.id));
                            this.lastTrailDrop = this.trailCreationCooldown;
                        }
                    }
                    else if (skillType === TYPE_CERRADA) {
                        if (this.cooldown <= 0) {
                            const margin = this.radius * 3;
                            const rx = margin + Math.random() * (canvas.width - margin * 2);
                            const ry = margin + Math.random() * (canvas.height - margin * 2);
                            closedSpaces.push(new ClosedSpace(rx, ry, this.id));
                            this.cooldown = 520; 
                        }
                    }
                    else if (skillType === TYPE_MAGNETICA) {
                        this.magnetTimer--;
                        if (this.magnetTimer <= 0) {
                            this.magnetPhase = this.magnetPhase === 'attract' ? 'repel' : 'attract';
                            this.magnetTimer = 180;
                        }

                        enemies.forEach(e => {
                            let dx = e.x - this.x;
                            let dy = e.y - this.y;
                            let dist = Math.hypot(dx, dy);

                            if (dist < this.magnetRange) {
                                let force = this.magnetPhase === 'attract' ? -0.12 : 0.35;
                                e.vx += (dx / dist) * force;
                                e.vy += (dy / dist) * force;

                                if (this.magnetPhase === 'repel' && this.magnetTimer % 60 === 0) {
                                    e.takeDamage(1);
                                }
                            }
                        });
                    }
                    else if (skillType === TYPE_FANTASMA) {
                        if (this.isGhostMode) {
                            this.ghostTimer--;
                            this.radius = BALL_RADIUS * 1.45; 
                            this.invulnTimer = 2; 
                            
                            if (this.ghostDamageCooldown > 0) this.ghostDamageCooldown--;
                            if (this.ghostDamageCooldown <= 0) {
                                enemies.forEach(e => {
                                    let dist = Math.hypot(this.x - e.x, this.y - e.y);
                                    if (dist < this.radius + e.radius) {
                                        e.takeDamage(3, this);
                                    }
                                });
                                this.ghostDamageCooldown = 30; 
                            }

                            if (this.ghostTimer <= 0) {
                                this.isGhostMode = false;
                                this.radius = BALL_RADIUS;
                                this.cooldown = 480; 
                            }
                        } else if (this.cooldown <= 0) {
                            this.isGhostMode = true;
                            this.ghostTimer = 300; 
                        }
                    }
                    else if (skillType === TYPE_ELECTRICA) {
                        if (this.cooldown <= 0) {
                            if (this.charges === 0) this.charges = 1; 

                            if (this.charges < 20) {
                                this.chargeTimer--;
                                if (this.chargeTimer <= 0) {
                                    this.charges++;
                                    this.chargeTimer = 120; 
                                }
                            }
                        } else {
                            this.charges = 0; 
                            this.chargeTimer = 120; 
                        }
                    }
                    else if (skillType === TYPE_HIELO) {
                        if (this.iceAuraActive) {
                            this.iceAuraTimer--;
                            if (this.iceAuraTimer <= 0) {
                                this.iceAuraActive = false; 
                                    this.iceCooldown = 300;
                                enemies.forEach(e => e.timeInIceAura = 0);
                            }

                            enemies.forEach(e => {
                                let dist = Math.hypot(this.x - e.x, this.y - e.y);
                                    if (dist < BALL_RADIUS * 3.57) {
                                    e.timeInIceAura = (e.timeInIceAura || 0) + 1;
                                    if (e.timeInIceAura >= 30) { 
                                        e.takeDamage(7, this);
                                        e.grabbedTimer = 120; 
                                        e.isFrozen = true;
                                        e.timeInIceAura = -60; 
                                        floatingTexts.push(new FloatingText(e.x, e.y - 40, "¡CONGELADO!", '#22d3ee'));
                                    }
                                    } else {
                                    e.timeInIceAura = 0;
                                }
                            });
                        }
                        else {
                            this.iceCooldown--;
                            if (this.iceCooldown <= 0) {
                                this.iceAuraActive = true;
                                this.iceAuraTimer = 240; 
                            }
                        }
                    }

                    if (skillType === TYPE_ALEATORIA) {
                        this.randomDamageTimer--;
                        if (this.randomDamageTimer <= 0) {
                            this.randomDamage = Math.floor(Math.random() * 10) + 1;
                            this.randomDamageTimer = 60; 
                        }
                    }

                    if (skillType === TYPE_MANZANA) {
                        if (this.cooldown <= 0) {
                            const rx = 50 + Math.random() * (canvas.width - 100);
                            const ry = 50 + Math.random() * (canvas.height - 100);
                            apples.push(new Apple(rx, ry, this.id));
                            this.cooldown = 140; 
                        }
                    }

                    if (skillType === TYPE_BOOMERANG) {
                        if (this.cooldown <= 0 && nearestEnemy) {
                            boomerangs.push(new Boomerang(this.x, this.y, this, nearestEnemy));
                            this.cooldown = 300; 
                            this.cooldown = 150; 
                        }
                    }

                    if (skillType === TYPE_ORBITAL) {
                        if (this.cooldown <= 0) {
                            const orbitRadius = this.radius * (1.5 + Math.random() * 2);
                            const orbitSpeed = (0.02 + Math.random() * 0.03) * (Math.random() > 0.5 ? 1 : -1);
                            orbitalPlanets.push(new OrbitalPlanet(this, orbitRadius, orbitSpeed));
                            this.cooldown = 300; 
                        }
                    }

                    if (skillType === TYPE_CAMPANA) {
                        let side = null;
                        if (this.x <= this.radius) side = 'left';
                        else if (this.x >= canvas.width - this.radius) side = 'right';
                        else if (this.y <= this.radius) side = 'top';
                        else if (this.y >= canvas.height - this.radius) side = 'bottom';

                        if (side && !this.lastHitWall) {
                            floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡DONG!", '#fde68a'));
                            dongEffects.push(new DongEffect(this.x, this.y));
                            ballsArray.forEach(b => { if (b.isAlive && b.id !== this.id) b.takeDamage(2, this); });
                        }
                        this.lastHitWall = !!side;
                    }

                    if (skillType === TYPE_SANDIA && this.isAlive && this.grabbedTimer <= 0) {
                        this.sandiaCooldown--;
                        if (this.sandiaCooldown <= 0) {
                            for(let i=0; i<2; i++) {
                                let ang = Math.random() * Math.PI * 2;
                                sandiaSeeds.push(new SandiaSeed(this.x, this.y, Math.cos(ang)*getEffectiveSpeed(6), Math.sin(ang)*getEffectiveSpeed(6), this.id, 2));
                            }
                            this.sandiaCooldown = 300;
                            floatingTexts.push(new FloatingText(this.x, this.y - 50, "¡SEMILLAS!", '#14532d'));
                        }
                    }

                    if (skillType === TYPE_GOLEM && this.isAlive && this.grabbedTimer <= 0) {
                        if (this.golemAttackTimer > 0) {
                            this.golemAttackTimer--;
                            // Solo hace daño en dos ráfagas: frames 60-50 y 30-20
                            const isAttacking = (this.golemAttackTimer > 50) || (this.golemAttackTimer < 30 && this.golemAttackTimer > 10);
                            if (isAttacking) {
                                enemies.forEach(e => {
                                    if (e.isAlive && !this.golemHitIds.has(e.id + "_" + (this.golemAttackTimer > 40 ? 1 : 2))) {
                                        if (Math.hypot(this.x - e.x, this.y - e.y) < this.radius * 1.6 + e.radius) {
                                            e.takeDamage(4, this);
                                            this.golemHitIds.add(e.id + "_" + (this.golemAttackTimer > 40 ? 1 : 2));
                                            floatingTexts.push(new FloatingText(e.x, e.y - 40, "¡MAZAZO!", '#78716c'));
                                        }
                                    }
                                });
                            }
                        } else {
                            this.golemCooldown--;
                            if (this.golemCooldown <= 0) {
                                this.golemAttackTimer = 65;
                                this.golemCooldown = 210;
                                this.golemHitIds.clear();
                            }
                        }
                    }

                    if (skillType === TYPE_BICHO) {
                        if (this.bichoAttackTimer > 0) {
                            this.bichoAttackTimer--;
                            
                            const stingerLen = this.radius * 2.8;
                            const isSpecial = this.bichoAttackCount % 3 === 0;

                            enemies.forEach(e => {
                                if (e.isAlive && !this.bichoHitIds.has(e.id)) {
                                    const tipX1 = this.x + Math.cos(this.bichoAttackAngle) * stingerLen;
                                    const tipY1 = this.y + Math.sin(this.bichoAttackAngle) * stingerLen;
                                    let d1 = Math.hypot(tipX1 - e.x, tipY1 - e.y);
                                    
                                    let hit = d1 < e.radius + 10;

                                    if (!hit && isSpecial) {
                                        const tipX2 = this.x + Math.cos(this.bichoAttackAngle + Math.PI) * stingerLen;
                                        const tipY2 = this.y + Math.sin(this.bichoAttackAngle + Math.PI) * stingerLen;
                                        let d2 = Math.hypot(tipX2 - e.x, tipY2 - e.y);
                                        hit = d2 < e.radius + 10;
                                    }

                                    if (hit) {
                                        e.takeDamage(15);
                                        e.applyPoison();
                                        this.bichoHitIds.add(e.id);
                                        floatingTexts.push(new FloatingText(e.x, e.y - 40, isSpecial ? "¡DOBLE PICADURA!" : "¡PICADURA!", '#ef4444'));
                                    }
                                }
                            });
                        } else if (this.cooldown <= 0) {
                            this.bichoAttackCount++;
                            this.bichoAttackAngle = Math.random() * Math.PI * 2;
                            this.bichoAttackTimer = 49;
                            this.bichoHitIds.clear();
                            this.cooldown = 120; 
                        }
                    }

                    if (skillType === TYPE_LASER_V2) {
                        let side = null;
                        if (this.x <= this.radius) side = 'left';
                        else if (this.x >= canvas.width - this.radius) side = 'right';
                        else if (this.y <= this.radius) side = 'top';
                        else if (this.y >= canvas.height - this.radius) side = 'bottom';

                        if (side && !this.lastHitWall) {
                            if (!this.laserAnchor) {
                                this.laserAnchor = { x: this.x, y: this.y };
                                floatingTexts.push(new FloatingText(this.x, this.y - 40, "ANCLAJE", '#22d3ee'));
                            } else {
                                laserBeamsV2.push(new LaserBeamV2(this.laserAnchor.x, this.laserAnchor.y, this.x, this.y, this.id));
                                this.laserAnchor = null;
                                floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡RAYO LÁSER!", '#22d3ee'));
                            }
                        }
                        this.lastHitWall = !!side;
                    }

                    if (skillType === TYPE_DESERTICA) {
                        if (this.desertTimer > 0) {
                            this.desertTimer--;
                            if (this.desertTimer % 60 === 0) {
                                enemies.forEach(e => e.takeDamage(1, this));
                            }
                            if (this.desertTimer === 0) {
                                this.cooldown = 630; 
                            }
                        } else if (this.cooldown <= 0) {
                            this.desertTimer = 360; 
                            for (let i = 0; i < 5; i++) {
                                let rx = 50 + Math.random() * (canvas.width - 100);
                                let ry = 50 + Math.random() * (canvas.height - 100);
                                cacti.push(new Cactus(rx, ry, this.id));
                            }
                            floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡TORMENTA DE ARENA!", '#c2b280'));
                        }
                    }

                    if (skillType === TYPE_MADERA) {
                        if (this.cooldown <= 0) {
                            const sides = ['left', 'right', 'top', 'bottom'];
                            const side = sides[Math.floor(Math.random() * sides.length)];
                            woodenPlanks.push(new WoodenPlank(side, this.id));
                            this.cooldown = 180; 
                        }
                    }

                    if (skillType === TYPE_CRISTAL) {
                        this.crystalAttractTimer--;
                        if (this.crystalAttractTimer <= 0) {
                            let shardsFound = false;
                            crystalShards.forEach(cs => {
                                if (cs.ownerId === this.id && cs.active) {
                                    cs.isBeingAttracted = true;
                                    cs.attractTarget = this;
                                    shardsFound = true;
                                }
                            });
                            if (shardsFound) {
                                floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡COSECHA!", '#f1f5f9'));
                            }
                            this.crystalAttractTimer = 600; 
                        }
                    }

                    if (skillType === TYPE_FURIA) {
                        if (!this.isFurious && this.hp <= 40) {
                            this.isFurious = true;
                            floatingTexts.push(new FloatingText(this.x, this.y - 50, "¡FURIA!", '#ef4444'));
                        }

                        if (this.isFurious) {
                            this.furyTimer = (this.furyTimer || 0) + 1;
                            if (this.furyTimer >= 30) {
                                this.takeDamage(1, null, false, true); 
                                this.furyTimer = 0;
                            }
                        }
                    }

                    if (skillType === TYPE_PROGRAMADORA) {
                        this.programDamageTimer--;
                        if (this.programDamageTimer <= 0) {
                            const text = DAMAGE_CODE_SNIPPETS[Math.floor(Math.random() * DAMAGE_CODE_SNIPPETS.length)];
                            const dmg = 2 + Math.random() * 4;
                            let vx = (Math.random() - 0.5) * 8;
                            let vy = (Math.random() - 0.5) * 8;
                            if (nearestEnemy) {
                                let angle = Math.atan2(nearestEnemy.y - this.y, nearestEnemy.x - this.x);
                                vx = Math.cos(angle) * getEffectiveSpeed(6);
                                vy = Math.sin(angle) * getEffectiveSpeed(6);
                            }
                            codeLines.push(new CodeLine(this.x, this.y, vx, vy, text, 'damage', this.id, dmg));
                            this.programDamageTimer = 240;
                        }
                        this.programHealTimer--;
                        if (this.programHealTimer <= 0) {
                            const text = HEAL_CODE_SNIPPETS[Math.floor(Math.random() * HEAL_CODE_SNIPPETS.length)];
                            const heal = 1 + Math.random() * 4;
                            let vx = (Math.random() - 0.5) * 4;
                            let vy = (Math.random() - 0.5) * 4;
                            codeLines.push(new CodeLine(this.x, this.y, vx, vy, text, 'heal', this.id, heal));
                            this.programHealTimer = 360;
                        }
                    }

                    if (skillType === TYPE_DRON) {
                        if (this.cooldown <= 0) {
                            drones.push(new Drone(this.x, this.y, this.id));
                            this.cooldown = 210; 
                        }
                    }

                    if (skillType === TYPE_TOPO) {
                        if (this.topoPhase > 0) {
                            this.invulnTimer = 2; 
                            
                            let dx = this.topoTargetX - this.x;
                            let dy = this.topoTargetY - this.y;
                            let dist = Math.hypot(dx, dy);

                            if (dist > 10) {
                                let speed = getEffectiveSpeed(15);
                                this.x += (dx / dist) * speed;
                                this.y += (dy / dist) * speed;

                                enemies.forEach(e => {
                                    if (!this.topoHitIds.has(e.id)) {
                                        if (Math.hypot(this.x - e.x, this.y - e.y) < this.radius + e.radius) {
                                            e.takeDamage(5, this);
                                            this.topoHitIds.add(e.id);
                                            floatingTexts.push(new FloatingText(e.x, e.y - 40, "¡GOLPE SUBTERRÁNEO!", '#78350f'));
                                        }
                                    }
                                });
                            } else {
                                this.topoPhase++;
                                this.topoHitIds.clear();
                                if (this.topoPhase > 3) {
                                    this.topoPhase = 0;
                                    this.cooldown = 420; 
                                } else {
                                    this.topoTargetX = Math.random() * (canvas.width - this.radius * 2) + this.radius;
                                    this.topoTargetY = Math.random() * (canvas.height - this.radius * 2) + this.radius;
                                }
                            }
                        } else if (this.cooldown <= 0) {
                            this.topoPhase = 1;
                            this.topoTargetX = Math.random() * (canvas.width - this.radius * 2) + this.radius;
                            this.topoTargetY = Math.random() * (canvas.height - this.radius * 2) + this.radius;
                            floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡BAJO TIERRA!", '#78350f'));
                        }
                    }

                    if (skillType === TYPE_MUERTE) {
                        this.soulSpawnTimer--;
                        if (this.soulSpawnTimer <= 0) {
                            const rx = 50 + Math.random() * (canvas.width - 100);
                            const ry = 50 + Math.random() * (canvas.height - 100);
                            darkSouls.push(new DarkSoul(rx, ry, this.id));
                            this.soulSpawnTimer = 180; 
                        }

                        this.deathAttackTimer--;
                        if (this.deathAttackTimer <= 0) {
                            if (this.soulsCollected > 0) {
                                const totalDamage = this.soulsCollected * 5;
                                enemies.forEach(e => {
                                    e.takeDamage(totalDamage, this);
                                    floatingTexts.push(new FloatingText(e.x, e.y - 60, `¡SENTENCIA! -${totalDamage}`, '#4c1d95'));
                                });
                                floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡LIBERACIÓN DE ALMAS!", '#a855f7'));
                                this.soulsCollected = 0;
                            } else {
                                floatingTexts.push(new FloatingText(this.x, this.y - 40, "SIN ALMAS...", '#6b7280'));
                            }
                            this.deathAttackTimer = 1800; 
                        }
                    }

                    if (skillType === TYPE_METALICA) {
                        if (this.cooldown <= 0) {
                            this.metalShotsLeft = 3;
                            this.metalShotTimer = 0; 
                            this.cooldown = 240; 
                        }

                        if (this.metalShotsLeft > 0) {
                            this.metalShotTimer--;
                            if (this.metalShotTimer <= 0 && nearestEnemy) {
                                let angle = Math.atan2(nearestEnemy.y - this.y, nearestEnemy.x - this.x);
                                projectiles.push(new Projectile(
                                    this.x + Math.cos(angle) * (this.radius + 10),
                                    this.y + Math.sin(angle) * (this.radius + 10),
                                    Math.cos(angle) * getEffectiveSpeed(PROYECTILE_SPEED),
                                    Math.sin(angle) * getEffectiveSpeed(PROYECTILE_SPEED),
                                    this.id,
                                    2 
                                ));
                                this.metalShotsLeft--;
                                this.metalShotTimer = 12; 
                            }
                        }
                    }

                    if (skillType === TYPE_DISCO && this.isAlive) {
                        this.discoCooldown--;
                        if (this.discoCooldown <= 0) {
                            let angle = Math.random() * Math.PI * 2;
                            let vx = Math.cos(angle) * getEffectiveSpeed(12);
                            let vy = Math.sin(angle) * getEffectiveSpeed(12);
                            discs.push(new Disc(this.x, this.y, vx, vy, this.id));
                            floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡DISCO LANZADO!", '#c084fc'));
                            this.discoCooldown = 330;
                            this.discoCooldown = 360;
                        }
                    }
                }
            }

            spawnSpiritualSouls(ballsArray) {
                let nearestEnemy = null;
                let minDist = Infinity;
                ballsArray.forEach(b => {
                    if (b.isAlive && b.id !== this.id && !b.isClone) {
                        let d = Math.hypot(this.x - b.x, this.y - b.y);
                        if (d < minDist) { minDist = d; nearestEnemy = b; }
                    }
                });
                if (nearestEnemy) {
                    const sides = [
                        {x: 0, y: Math.random() * canvas.height},
                        {x: canvas.width, y: Math.random() * canvas.height},
                        {x: Math.random() * canvas.width, y: 0},
                        {x: Math.random() * canvas.width, y: canvas.height}
                    ];
                const pos = sides[Math.floor(Math.random() * sides.length)];
                let dx = nearestEnemy.x - pos.x;
                let dy = nearestEnemy.y - pos.y;
                let dist = Math.hypot(dx, dy);
                let speed = getEffectiveSpeed(15);
                spiritualSouls.push(new SpiritualSoul(pos.x, pos.y, (dx / dist) * speed, (dy / dist) * speed, this.id));
                }
            }

            startSlipping() {
                this.slipTimer = 90; 
                floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡RESBALA!", '#0ea5e9'));
            }

            applyBurn() {
                this.burnTimer = 180; 
                this.burnDamageCooldown = 0; 
                floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡QUEMADO!", '#f97316'));
            }

            applyPoison() {
                this.poisonTimer = 180; 
                this.poisonDamageCooldown = 60; 
                floatingTexts.push(new FloatingText(this.x, this.y - 60, "¡ENVENENADO!", '#22c55e'));
            }

         
            takeDamage(amount, attacker = null, isReflected = false, isFuryDecay = false, ignoreInvuln = false) {
                const skillType = (this.type === TYPE_ERROR && this.currentErrorType) ? this.currentErrorType : this.type;
                
                if (skillType === TYPE_ESPIRITUAL && this.spiritualTimer > 0 && !ignoreInvuln) {
                    this.spiritualTimer -= 60; // Pierde 1 segundo
                    this.spawnSpiritualSouls(balls); 
                    this.spiritualStunTimer = 120; // 2 segundos sin atacar automáticamente
                    this.spiritualStunTimer = 84; // 1.4 segundos (84 frames / 60fps)
                    return;
                }

                if (!ignoreInvuln && (this.type === TYPE_PRUEBA || this.invulnTimer > 0 || (skillType === TYPE_DESLIZANTE && this.slideTimer > 0) || (skillType === TYPE_GRAVITACIONAL && this.gravityTimer > 0) || (skillType === TYPE_TIEMPO && this.timeStopTimer > 0))) return;
                
                if (skillType === TYPE_FURIA && this.isFurious && !isFuryDecay && !ignoreInvuln) return;

                if (skillType === TYPE_ESCUDERA && amount > 0) {
                    this.pendingCounterDamage += amount;
                }

                if (skillType === TYPE_ESCUDERA && attacker && attacker.isAlive && !isReflected && amount > 0) {
                    attacker.takeDamage(amount, this, true, false, true); 
                    floatingTexts.push(new FloatingText(this.x, this.y - 65, "¡CONTRATAQUE!", '#cbd5e1'));
                }

                let damageToHp = amount;
                if (skillType === TYPE_METALICA) damageToHp *= 0.75;
                if (skillType === TYPE_ABSORVEDORA) {
                    this.absorbedDamage += amount; // Absorbe el 100% del impacto original
                    damageToHp *= 0.5; // Pero solo recibe el 50% en su vida
                }
                if (skillType === TYPE_HOMBRELOBO && this.wolfTimer > 0) damageToHp *= 0.5;

                if (skillType === TYPE_CRISTAL && amount > 0) {
                if (this.weaknessTimer > 0) damageToHp *= 0.8; // Efecto Pelota Color Azul

                    for (let i = 0; i < 8; i++) {
                        const angle = Math.random() * Math.PI * 2;
                        const speed = 4 + Math.random() * 6;
                        crystalShards.push(new CrystalShard(this.x, this.y, Math.cos(angle) * speed, Math.sin(angle) * speed, this.id));
                    }
                    floatingTexts.push(new FloatingText(this.x, this.y - 50, "¡CRISTAL ROTO!", '#f1f5f9'));
                }

                if (skillType === TYPE_ESCUDERA && this.shieldHp > 0 && !isReflected) {
                    let absorbed = Math.min(amount, this.shieldHp);
                    this.shieldHp -= absorbed;
                    damageToHp -= absorbed;
                    floatingTexts.push(new FloatingText(this.x, this.y - 45, `🛡️ -${Math.ceil(absorbed)}`, '#0ea5e9'));

                    if (this.shieldHp <= 0) {
                        this.shieldCooldown = 600; 
                    }
                }

                if (damageToHp <= 0) return;

                this.hp -= damageToHp;
                if (this.hp <= 0 && this.isAlive) {
                    if (skillType === TYPE_DOBLE && this.deathTimer === 0) {
                        this.deathTimer = 60; // 1 segundo a 60fps
                        this.vx = 0; this.vy = 0;
                        floatingTexts.push(new FloatingText(this.x, this.y - 40, "¡ÚLTIMO ADIÓS!", '#7dd3fc'));
                        return;
                    }
                }

                const dmgDisplay = damageToHp < 1 ? damageToHp.toFixed(1) : Math.ceil(damageToHp);
                floatingTexts.push(new FloatingText(this.x, this.y - 20, `-${dmgDisplay}`, '#ef4444'));
                this.invulnTimer = 15; 
                
                if (this.hp <= 0 && this.isAlive) {
                    this.hp = 0;
                    this.isAlive = false;
                    
                    if (!this.isClone) {
                        ranking.push(this);
                    }
                    
                    if (this.isClone && this.cloneOwner && this.cloneOwner.isAlive) {
                        this.cloneOwner.takeDamage(5);
                        this.cloneOwner.activeClone = null;
                    }
                }
            }

            draw(ctx) {
                if (!this.isAlive) return;
                const skillType = (this.type === TYPE_ERROR && this.currentErrorType) ? this.currentErrorType : this.type;

                if (skillType === TYPE_SOL && this.isAlive) {
                    if (this.solUltActive) {
                        ctx.save();
                        ctx.globalAlpha = 0.2 + Math.sin(Date.now() / 100) * 0.1;
                        ctx.fillStyle = '#f97316';
                        ctx.fillRect(0, 0, canvas.width, canvas.height);
                        ctx.restore();
                    } else if (this.solActive) {
                        ctx.save();
                        ctx.beginPath();
                        const grad = ctx.createRadialGradient(this.x, this.y, this.radius, this.x, this.y, this.solRange);
                        grad.addColorStop(0, 'rgba(250, 204, 21, 0.4)');
                        grad.addColorStop(1, 'rgba(234, 88, 12, 0)');
                        ctx.fillStyle = grad;
                        ctx.arc(this.x, this.y, this.solRange, 0, Math.PI * 2);
                        ctx.fill();
                        ctx.strokeStyle = 'rgba(251, 191, 36, 0.3)';
                        ctx.setLineDash([5, 5]);
                        ctx.stroke();
                        ctx.restore();
                    }
                }

                if (skillType === TYPE_ABSORVEDORA && this.isAlive) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 5 + Math.sin(Date.now()/200)*3, 0, Math.PI * 2);
                    const alpha = 0.2 + (this.absorbedDamage / 200); // Brilla más cuanto más daño tiene
                    ctx.strokeStyle = `rgba(16, 185, 129, ${Math.min(0.8, alpha)})`;
                    ctx.lineWidth = 3;
                    ctx.stroke();
                    ctx.restore();
                }

                if (skillType === TYPE_CUESTIONADORA && this.questionTimer > 0 && this.isAlive) {
                    ctx.save();
                    const midX = canvas.width / 2;
                    
                    // Línea divisoria
                    ctx.strokeStyle = 'rgba(99, 102, 241, 0.4)';
                    ctx.lineWidth = 4;
                    ctx.setLineDash([10, 10]);
                    ctx.beginPath();
                    ctx.moveTo(midX, 0); ctx.lineTo(midX, canvas.height);
                    ctx.stroke();

                    // Pregunta
                    ctx.fillStyle = 'white';
                    ctx.font = 'bold 24px Arial';
                    ctx.textAlign = 'center';
                    ctx.shadowBlur = 10;
                    ctx.shadowColor = '#000';
                    ctx.fillText("¿Puedo hacerte daño?", midX, 50);

                    // Opciones Sí / No
                    ctx.font = 'bold 60px Arial';
                    const leftText = this.questionSide === 'left' ? "SÍ" : "NO";
                    const rightText = this.questionSide === 'right' ? "SÍ" : "NO";
                    
                    ctx.globalAlpha = 0.3;
                    ctx.fillStyle = this.questionSide === 'left' ? '#ef4444' : '#22c55e';
                    ctx.fillText(leftText, midX / 2, canvas.height / 2);
                    
                    ctx.fillStyle = this.questionSide === 'right' ? '#ef4444' : '#22c55e';
                    ctx.fillText(rightText, midX + midX / 2, canvas.height / 2);
                    
                    ctx.restore();
                }

                if (skillType === TYPE_TOPO && this.topoPhase > 0) {
                    ctx.globalAlpha = 0.3; 
                }

                if (this.isDashing || this.launchedTimer > 0) {
                    ctx.beginPath();
                    ctx.arc(this.x - this.vx, this.y - this.vy, this.radius, 0, Math.PI * 2);
                    ctx.fillStyle = 'rgba(252, 165, 165, 0.4)';
                    ctx.fill();
                    ctx.closePath();
                }

                if (skillType === TYPE_COLOR && this.isAlive) {
                    ctx.save();
                    this.activeColors.forEach((colName, idx) => {
                        const colorMap = { rojo: '#ef4444', verde: '#22c55e', amarillo: '#facc15', morado: '#a855f7', azul: '#3b82f6', negro: '#000000' };
                        ctx.beginPath();
                        ctx.arc(this.x, this.y, this.radius + 6 + (idx * 5), 0, Math.PI * 2);
                        ctx.strokeStyle = colorMap[colName] || '#fff';
                        ctx.lineWidth = 3;
                        ctx.setLineDash([5, 3]);
                        ctx.lineDashOffset = (Date.now() / 20) * (idx === 0 ? 1 : -1);
                        ctx.stroke();
                    });
                    ctx.restore();
                }

                if (skillType === TYPE_FANTASMA && this.isGhostMode) ctx.globalAlpha = 0.4;

                if (skillType === TYPE_ESPIRITUAL && this.isAlive) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 5 + Math.sin(Date.now()/100)*2, 0, Math.PI * 2);
                    ctx.strokeStyle = this.spiritualTimer > 0 ? 'rgba(129, 140, 248, 0.6)' : 'rgba(239, 68, 68, 0.6)';
                    ctx.setLineDash([5, 5]);
                    ctx.lineWidth = 2;
                    ctx.stroke();
                    ctx.restore();
                }

                if (skillType === TYPE_APOSTADORA && (this.jackpotHealTimer > 0 || this.jackpotDamageTimer > 0)) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 10, 0, Math.PI * 2);
                    ctx.strokeStyle = '#fbbf24';
                    ctx.lineWidth = 4;
                    ctx.setLineDash([5, 5]);
                    ctx.stroke();
                    ctx.restore();
                }

                if (skillType === TYPE_CONEJO && this.isAlive) {
                    ctx.save();
                    ctx.translate(this.x, this.y);
                    ctx.fillStyle = this.color;
                    ctx.strokeStyle = this.borderColor;
                    ctx.lineWidth = 2;
                    ctx.beginPath();
                    ctx.ellipse(-15, -this.radius + 5, 8, 20, -0.2, 0, Math.PI * 2);
                    ctx.fill(); ctx.stroke();
                    ctx.beginPath();
                    ctx.ellipse(15, -this.radius + 5, 8, 20, 0.2, 0, Math.PI * 2);
                    ctx.fill(); ctx.stroke();
                    ctx.restore();
                }

                if (skillType === TYPE_NUMERICA && this.numericTimer > 0 && this.isAlive) {
                    ctx.save();
                    const w = canvas.width / 3;
                    const h = canvas.height / 3;
                    ctx.strokeStyle = 'rgba(34, 211, 238, 0.2)';
                    ctx.lineWidth = 2;
                    ctx.font = 'bold 30px Arial';
                    ctx.textAlign = 'center';
                    for (let i = 0; i < 9; i++) {
                        let r = Math.floor(i / 3);
                        let c = i % 3;
                        ctx.strokeRect(c * w, r * h, w, h);
                        ctx.fillStyle = 'rgba(34, 211, 238, 0.4)';
                        ctx.fillText(this.numericGrid[i], c * w + w / 2, r * h + h / 2 + 10);
                    }
                    ctx.restore();
                }

                if (skillType === TYPE_TOPO && this.topoPhase > 0) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 5, 0, Math.PI * 2);
                    ctx.strokeStyle = '#451a03';
                    ctx.lineWidth = 2;
                    ctx.setLineDash([3, 3]);
                    ctx.stroke();
                    ctx.restore();
                }

                if (skillType === TYPE_FURIA && this.isFurious) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 5 + Math.sin(Date.now() / 50) * 5, 0, Math.PI * 2);
                    ctx.strokeStyle = '#ef4444';
                    ctx.lineWidth = 3;
                    ctx.stroke();
                    ctx.restore();
                }
                
                if (skillType === TYPE_DESLIZANTE && this.slideTimer > 0) {
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 5, 0, Math.PI * 2);
                    ctx.strokeStyle = '#2dd4bf';
                    ctx.lineWidth = 2;
                    ctx.stroke();
                }

                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.lineWidth = 3;
                ctx.strokeStyle = this.borderColor;
                ctx.stroke();

                if (this.type === TYPE_ERROR) {
                    ctx.save();
                    ctx.globalAlpha = 0.4;
                    for(let i=0; i<3; i++) {
                        let ox = (Math.random()-0.5)*15;
                        let oy = (Math.random()-0.5)*15;
                        ctx.strokeStyle = i === 0 ? '#00ffff' : '#ff00ff';
                        ctx.beginPath();
                        ctx.arc(this.x + ox, this.y + oy, this.radius, 0, Math.PI*2);
                        ctx.stroke();
                    }
                    ctx.restore();
                }
                ctx.closePath();

                if (skillType === TYPE_GRAVITACIONAL && this.gravityTimer > 0) {
                    ctx.save();
                    const pulse = Math.sin(Date.now() / 100) * 10;
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 15 + pulse, 0, Math.PI * 2);
                    ctx.strokeStyle = '#38bdf8';
                    ctx.lineWidth = 4;
                    ctx.stroke();
                    
                    balls.forEach(e => {
                        if (e.isAlive && e.gravityPushTimer > 0) {
                            ctx.beginPath();
                            ctx.arc(e.x, e.y, e.radius + 10 + pulse, 0, Math.PI * 2);
                            ctx.strokeStyle = 'rgba(56, 189, 248, 0.5)';
                            ctx.stroke();
                        }
                    });
                    ctx.restore();
                }

                if (skillType === TYPE_DIVISORA && this.divisoraTimer > 0) {
                    ctx.save();
                    const midX = canvas.width / 2;
                    const midY = canvas.height / 2;

                    ctx.strokeStyle = 'rgba(239, 68, 68, 0.4)';
                    ctx.lineWidth = 4;
                    ctx.beginPath();
                    ctx.moveTo(midX, 0); ctx.lineTo(midX, canvas.height);
                    ctx.moveTo(0, midY); ctx.lineTo(canvas.width, midY);
                    ctx.stroke();

                    ctx.fillStyle = 'rgba(239, 68, 68, 0.2)';
                    let qx = (this.divisoraQuadrant % 2 === 1) ? midX : 0;
                    let qy = (this.divisoraQuadrant >= 2) ? midY : 0;
                    ctx.fillRect(qx, qy, midX, midY);
                    ctx.restore();
                }

                if (skillType === TYPE_TIEMPO && this.timeStopTimer > 0) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 15, 0, Math.PI * 2);
                    ctx.strokeStyle = '#6366f1';
                    ctx.lineWidth = 4;
                    ctx.setLineDash([10, 5]);
                    ctx.lineDashOffset = -(Date.now() / 30) % 15;
                    ctx.stroke();
                    
                    ctx.moveTo(this.x, this.y);
                    ctx.lineTo(this.x + Math.cos(Date.now()/200)*this.radius, this.y + Math.sin(Date.now()/200)*this.radius);
                    ctx.stroke();
                    ctx.restore();
                }

                if (skillType === TYPE_RAYO && this.rayoStrikeStage === 1 && this.isAlive) {
                    const isTriple = (this.rayoStrikeCount % 3 === 0);
                    const strikeRadius = isTriple ? this.radius * 1.4 : this.radius * 2;
                    this.rayoStrikes.forEach(s => {
                        ctx.save();
                        ctx.beginPath();
                        ctx.arc(s.x, s.y, strikeRadius, 0, Math.PI * 2);
                        const pulse = 0.2 + Math.sin(Date.now() / 100) * 0.1;
                        ctx.fillStyle = `rgba(253, 224, 71, ${pulse})`;
                        ctx.fill();
                        ctx.strokeStyle = '#fde047';
                        ctx.lineWidth = 2;
                        ctx.setLineDash([5, 5]);
                        ctx.stroke();
                        ctx.restore();
                    });
                }

                if (skillType === TYPE_PIEDRA && this.isPetrified && this.isAlive) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius * 1.5, 0, Math.PI * 2);
                    const pulse = 0.1 + Math.sin(Date.now() / 200) * 0.05;
                    ctx.fillStyle = `rgba(120, 113, 108, ${pulse})`;
                    ctx.fill();
                    ctx.strokeStyle = 'rgba(120, 113, 108, 0.5)';
                    ctx.lineWidth = 2;
                    ctx.stroke();
                    ctx.restore();
                }

                ctx.fillStyle = '#000';
                ctx.font = 'bold 14px Arial';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(this.isClone ? `${this.name}C` : this.name, this.x, this.y - 8);

                ctx.font = 'bold 12px Arial';
                ctx.fillText(`${Math.ceil(this.hp)}/100`, this.x, this.y + 8);

                if (skillType === TYPE_ALEATORIA) {
                    ctx.font = 'bold 11px Arial';
                    ctx.fillText(`Daño: ${this.randomDamage}`, this.x, this.y + 22);
                }

                const barWidth = 50;
                const barHeight = 6;
                const healthRatio = Math.max(0, this.hp / this.maxHp);
                
                ctx.fillStyle = '#ef4444'; 
                ctx.fillRect(this.x - barWidth/2, this.y - this.radius - 12, barWidth, barHeight);
                ctx.fillStyle = '#22c55e'; 
                ctx.fillRect(this.x - barWidth/2, this.y - this.radius - 12, barWidth * healthRatio, barHeight);
                ctx.strokeStyle = '#000';
                ctx.lineWidth = 1;
                ctx.strokeRect(this.x - barWidth/2, this.y - this.radius - 12, barWidth, barHeight);

                if (skillType === TYPE_VAMPIRO) {
                    ctx.fillStyle = '#ffffff'; 
                    const fangSize = 25; 
                    let fangAngle = 0;
                    if (Math.hypot(this.vx, this.vy) > 0.1) {
                        fangAngle = Math.atan2(this.vy, this.vx);
                    } else {
                        fangAngle = Math.PI / 2;
                    }

                    ctx.beginPath();
                    ctx.moveTo(this.x + Math.cos(fangAngle - 0.5) * this.radius, this.y + Math.sin(fangAngle - 0.5) * this.radius);
                    ctx.lineTo(this.x + Math.cos(fangAngle - 0.35) * (this.radius + fangSize), this.y + Math.sin(fangAngle - 0.35) * (this.radius + fangSize));
                    ctx.lineTo(this.x + Math.cos(fangAngle - 0.2) * this.radius, this.y + Math.sin(fangAngle - 0.2) * this.radius);
                    ctx.moveTo(this.x + Math.cos(fangAngle + 0.2) * this.radius, this.y + Math.sin(fangAngle + 0.2) * this.radius);
                    ctx.lineTo(this.x + Math.cos(fangAngle + 0.35) * (this.radius + fangSize), this.y + Math.sin(fangAngle + 0.35) * (this.radius + fangSize));
                    ctx.lineTo(this.x + Math.cos(fangAngle + 0.5) * this.radius, this.y + Math.sin(fangAngle + 0.5) * this.radius);
                    ctx.fill();
                    ctx.closePath();
                }
                
                if (skillType === TYPE_LASER && this.laserTarget) {
                    ctx.beginPath();
                    ctx.moveTo(this.x, this.y);
                    ctx.lineTo(this.laserTarget.x, this.laserTarget.y);
                    ctx.strokeStyle = '#ff4500'; 
                    ctx.lineWidth = 3;
                    ctx.stroke();
                    ctx.closePath();
                }
                
                if (skillType === TYPE_ESPADACHINA) {
                    const swordX = this.x + Math.cos(this.swordAngle) * this.swordOrbitRadius;
                    const swordY = this.y + Math.sin(this.swordAngle) * this.swordOrbitRadius;
                    const swordTipX = swordX + Math.cos(this.swordAngle + Math.PI / 2) * this.swordLength; 
                    const swordTipY = swordY + Math.sin(this.swordAngle + Math.PI / 2) * this.swordLength;

                ctx.beginPath();
                ctx.moveTo(swordX, swordY);
                ctx.lineTo(swordTipX, swordTipY);
                ctx.strokeStyle = '#a9a9a9'; 
                ctx.lineWidth = 4;
                ctx.stroke();
                ctx.closePath();
                }

                if (this.burnTimer > 0) {
                    ctx.save();
                    ctx.globalAlpha = 0.6;
                    ctx.fillStyle = '#f97316';
                    for(let i=0; i<5; i++) {
                        let ox = (Math.random() - 0.5) * this.radius * 2;
                        let oy = (Math.random() - 0.5) * this.radius * 2;
                        ctx.beginPath();
                        ctx.arc(this.x + ox, this.y + oy, 5, 0, Math.PI*2);
                        ctx.fill();
                    }
                    ctx.restore();
                }

                if (this.poisonTimer > 0) {
                    ctx.save();
                    ctx.globalAlpha = 0.6;
                    ctx.fillStyle = '#22c55e';
                    for(let i=0; i<3; i++) {
                        let ox = (Math.random() - 0.5) * this.radius * 1.5;
                        let oy = (Math.random() - 0.5) * this.radius * 1.5;
                        ctx.beginPath();
                        ctx.arc(this.x + ox, this.y + oy, 4, 0, Math.PI*2);
                        ctx.fill();
                    }
                    ctx.restore();
                }

                if (this.isTrappedByJelly && this.isAlive) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 8, 0, Math.PI * 2);
                    ctx.fillStyle = 'rgba(244, 114, 182, 0.5)'; 
                    ctx.fill();
                    ctx.strokeStyle = '#f472b6';
                    ctx.lineWidth = 3;
                    ctx.stroke();
                    ctx.restore();
                }

                if (this.isFrozen && this.isAlive) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 5, 0, Math.PI * 2);
                    ctx.fillStyle = 'rgba(165, 243, 252, 0.4)'; 
                    ctx.fill();
                    ctx.strokeStyle = '#22d3ee';
                    ctx.lineWidth = 3;
                    ctx.stroke();
                    ctx.restore();
                }

                if (skillType === TYPE_HIELO && this.iceAuraActive && this.isAlive) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, BALL_RADIUS * 2.55, 0, Math.PI * 2); 
                    ctx.arc(this.x, this.y, BALL_RADIUS * 3.08, 0, Math.PI * 2); 
                    ctx.arc(this.x, this.y, BALL_RADIUS * 3.57, 0, Math.PI * 2);
                    const pulse = Math.sin(Date.now() / 150) * 0.1 + 0.2;
                    ctx.fillStyle = `rgba(34, 211, 238, ${pulse})`;
                    ctx.fill();
                    ctx.strokeStyle = 'rgba(34, 211, 238, 0.5)';
                    ctx.setLineDash([10, 5]);
                    ctx.stroke();
                    ctx.restore();
                }

                if (skillType === TYPE_CARNIVORA && this.isAlive) {
                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius * 2.2, 0, Math.PI * 2);
                    ctx.strokeStyle = this.carnivoreCooldown <= 0 ? 'rgba(190, 24, 93, 0.3)' : 'rgba(156, 163, 175, 0.1)';
                    ctx.setLineDash([5, 10]);
                    ctx.lineWidth = 2;
                    ctx.stroke();
                    ctx.fillStyle = this.carnivoreCooldown <= 0 ? 'rgba(190, 24, 93, 0.05)' : 'transparent';
                    ctx.fill();
                    ctx.restore();

                    ctx.save();
                    ctx.translate(this.x, this.y);
                    ctx.rotate(Date.now() / 200);
                    ctx.fillStyle = '#4a044e';
                    for(let i=0; i<4; i++) {
                        ctx.rotate(Math.PI / 2);
                        ctx.beginPath();
                        ctx.moveTo(this.radius - 5, -10);
                        ctx.lineTo(this.radius + 15, 0);
                        ctx.lineTo(this.radius - 5, 10);
                        ctx.fill();
                    }
                    ctx.restore();
                }

                if (skillType === TYPE_BOMBA && this.bombActive && this.bombCarrier && this.bombCarrier.isAlive) {
                    ctx.save();
                    const carrier = this.bombCarrier;
                    const bX = carrier.x;
                    const bY = carrier.y - carrier.radius - 15;
                    
                    ctx.beginPath();
                    ctx.arc(bX, bY, 12, 0, Math.PI * 2);
                    ctx.fillStyle = '#111827';
                    ctx.fill();
                    ctx.strokeStyle = '#fff';
                    ctx.lineWidth = 1;
                    ctx.stroke();
                    
                    ctx.beginPath();
                    ctx.moveTo(bX, bY - 12);
                    ctx.lineTo(bX + 8, bY - 20);
                    ctx.strokeStyle = '#f59e0b';
                    ctx.lineWidth = 2;
                    ctx.stroke();
                    ctx.fillStyle = (Date.now() % 200 < 100) ? '#fbbf24' : '#ef4444';
                    ctx.beginPath();
                    ctx.arc(bX + 8, bY - 20, 4, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.restore();
                }

                if (skillType === TYPE_PROYECCION && this.projectionTimer > 0 && this.isAlive) {
                    ctx.save();
                    ctx.setLineDash([10, 5]);
                    ctx.strokeStyle = 'rgba(129, 140, 248, 0.6)';
                    ctx.lineWidth = 3;
                    ctx.beginPath();
                    ctx.moveTo(this.x, this.y);
                    this.projectionPath.forEach(p => ctx.lineTo(p.x, p.y));
                    ctx.stroke();
                    this.projectionPath.forEach((p, i) => {
                        ctx.fillStyle = 'rgba(129, 140, 248, 0.8)';
                        ctx.fillText(i + 1, p.x, p.y - 15);
                    });
                    ctx.restore();
                }

                if (skillType === TYPE_AGUA) {
                    ctx.save();
                    ctx.fillStyle = 'rgba(14, 165, 233, 0.6)';
                    for(let i=0; i<3; i++) {
                        let offset = Math.sin((Date.now() / 200) + i) * 5;
                        ctx.beginPath();
                        ctx.arc(this.x - 15 + (i*15), this.y + this.radius - 5 + offset, 4, 0, Math.PI*2);
                        ctx.fill();
                    }
                    ctx.restore();
                }

                if (skillType === TYPE_ELECTRICA && this.charges > 0) {
                    ctx.save();
                    ctx.strokeStyle = '#fef08a';
                    ctx.lineWidth = 2;
                    for (let i = 0; i < this.charges; i++) {
                        const angle = (i / this.charges) * Math.PI * 2 + (Date.now() / 150);
                        ctx.beginPath();
                        ctx.moveTo(this.x + Math.cos(angle) * this.radius, this.y + Math.sin(angle) * this.radius);
                        ctx.lineTo(this.x + Math.cos(angle) * (this.radius + 10), this.y + Math.sin(angle) * (this.radius + 10));
                        ctx.stroke();
                    }
                    ctx.restore();
                }

                if (skillType === TYPE_MAGNETICA) {
                    const isAttracting = this.magnetPhase === 'attract';
                    const dashOffset = (Date.now() / 20) % 20; 
                    
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.magnetRange, 0, Math.PI * 2);
                    ctx.strokeStyle = isAttracting ? 'rgba(56, 189, 248, 0.5)' : 'rgba(239, 68, 68, 0.5)';
                    
                    ctx.setLineDash([10, 10]);
                    ctx.lineDashOffset = isAttracting ? dashOffset : -dashOffset;
                    
                    ctx.lineWidth = 4;
                    ctx.stroke();
                    ctx.setLineDash([]);
                    
                    ctx.globalAlpha = 0.1;
                    ctx.fillStyle = isAttracting ? '#38bdf8' : '#ef4444';
                    ctx.fill();
                    ctx.globalAlpha = 1.0;
                }

                if (skillType === TYPE_CAMPANA && this.isAlive) {
                    ctx.save();
                    ctx.translate(this.x, this.y);
                    ctx.rotate(Math.sin(Date.now() / 150) * 0.3); 
                    ctx.fillStyle = '#fbbf24';
                    ctx.beginPath();
                    ctx.arc(this.radius - 5, -this.radius, 10, Math.PI, 0); 
                    ctx.rect(this.radius - 15, -this.radius, 20, 15); 
                    ctx.fill();
                    ctx.beginPath();
                    ctx.arc(this.radius - 5, -this.radius + 15, 4, 0, Math.PI * 2); 
                    ctx.fillStyle = '#78350f';
                    ctx.fill();
                    ctx.restore();
                }

                if (skillType === TYPE_DISCO && this.isAlive && this.discoCooldown <= 60 && this.discoCooldown > 0) {
                    ctx.save();
                    ctx.translate(this.x, this.y);
                    ctx.rotate(Date.now() / 50);
                    ctx.beginPath();
                    ctx.arc(0, 0, this.radius * 0.7, 0, Math.PI * 2);
                    ctx.arc(0, 0, this.radius * 1.1, 0, Math.PI * 2);
                    ctx.strokeStyle = '#c084fc';
                    ctx.lineWidth = 4;
                    ctx.stroke();
                    ctx.fillStyle = '#fdf2f8';
                    for(let i=0; i<4; i++) {
                        ctx.rotate(Math.PI/2);
                        ctx.fillRect(this.radius*0.3, -2, this.radius*0.3, 4);
                        ctx.fillRect(this.radius * 0.7, -2, this.radius * 0.4, 4);
                    }
                    ctx.restore();
                }

                if (skillType === TYPE_GOLEM && this.isAlive && this.golemAttackTimer > 0) {
                    ctx.save();
                    const isPulse = (this.golemAttackTimer > 50) || (this.golemAttackTimer < 30 && this.golemAttackTimer > 10);
                    if (isPulse) {
                        ctx.beginPath();
                        ctx.arc(this.x, this.y, this.radius * 1.6, 0, Math.PI * 2);
                        ctx.fillStyle = 'rgba(120, 113, 108, 0.3)';
                        ctx.fill();
                        ctx.strokeStyle = '#44403c';
                        ctx.setLineDash([5, 5]);
                        ctx.lineWidth = 2;
                        ctx.stroke();
                    }
                    ctx.restore();
                }

                if (skillType === TYPE_PORTAL && this.isAlive) {
                    if (this.portalCooldown <= 30 && this.portalNextX !== null) {
                        ctx.save();
                        ctx.beginPath();
                        ctx.arc(this.portalNextX, this.portalNextY, this.teleportDamageRadius, 0, Math.PI * 2);
                        const alpha = (30 - this.portalCooldown) / 60; 
                        ctx.fillStyle = `rgba(139, 92, 246, ${alpha})`;
                        ctx.fill();
                        ctx.strokeStyle = `rgba(139, 92, 246, 0.5)`;
                        ctx.setLineDash([5, 5]);
                        ctx.stroke();
                        ctx.restore();
                    }

                    ctx.save();
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius + 5 + Math.sin(Date.now() / 100) * 3, 0, Math.PI * 2);
                    ctx.strokeStyle = '#8b5cf6'; 
                    ctx.lineWidth = 4;
                    ctx.setLineDash([5, 5]);
                    ctx.stroke();
                    ctx.setLineDash([]);
                    ctx.restore();
                }

                if (skillType === TYPE_BICHO && this.isAlive && this.bichoAttackTimer > 0) {
                    const isSpecial = this.bichoAttackCount % 3 === 0;
                    const angles = [this.bichoAttackAngle];
                    if (isSpecial) angles.push(this.bichoAttackAngle + Math.PI);

                    angles.forEach(angle => {
                        ctx.save();
                        ctx.translate(this.x, this.y);
                        ctx.rotate(angle);
                        ctx.beginPath();
                        const extend = Math.sin((this.bichoAttackTimer / 49) * Math.PI) * (this.radius * 1.4);
                        ctx.moveTo(this.radius - 5, -9.2); 
                        ctx.lineTo(this.radius + 14 + extend, 0); 
                        ctx.lineTo(this.radius - 5, 10);
                        ctx.fillStyle = isSpecial ? '#ef4444' : '#1a2e05'; 
                        ctx.fill();
                        ctx.strokeStyle = '#facc15';
                        ctx.lineWidth = 2;
                        ctx.stroke();
                        ctx.restore();
                    });
                }

                ctx.globalAlpha = 1.0;
            }
        }
        function updatePlayerConfigs() {
            const num = parseInt(numBallsSelect.value);
            const searchTerm = (document.getElementById('searchBall')?.value || "").toLowerCase();
            
            const currentSelections = [];
            for (let i = 1; i <= 4; i++) {
                const n = document.getElementById(`name_${i}`);
                const t = document.getElementById(`type_${i}`);
                const c = document.getElementById(`ctrl_${i}`);
                if (n && t && c) currentSelections[i] = { name: n.value, type: t.value, ctrl: c.value };
            }

            playerConfigsDiv.innerHTML = '';
            
            const filteredTypes = ALL_BALL_TYPES.filter(t => t.label.toLowerCase().includes(searchTerm));

            for (let i = 1; i <= num; i++) {
                const config = currentSelections[i] || { name: `J${i}`, type: 'random_pick', ctrl: 'player' };
                
                let optionsHtml = filteredTypes.map(t => 
                    `<option value="${t.value}" ${t.value === config.type ? 'selected' : ''}>${t.label}</option>`
                ).join('');
                
                if (config.type && !filteredTypes.find(t => t.value === config.type)) {
                    const original = ALL_BALL_TYPES.find(t => t.value === config.type);
                    if (original) optionsHtml = `<option value="${original.value}" selected>${original.label}</option>` + optionsHtml;
                }

                playerConfigsDiv.innerHTML += `
                    <div class="flex items-center gap-2 bg-gray-700 p-2 rounded-lg">
                        <input id="name_${i}" type="text" value="${config.name}" class="bg-gray-800 text-white p-1 rounded w-16 border border-gray-600 focus:outline-none focus:border-blue-500 text-center font-bold text-sm" placeholder="Nombre">
                        <select id="type_${i}" class="bg-gray-800 text-white p-1 rounded flex-1 border border-gray-600 text-sm">
                            ${optionsHtml}
                        </select>
                        <button onclick="document.getElementById('type_${i}').value = 'random_pick'" class="bg-blue-600 hover:bg-blue-500 text-white px-2 py-1 rounded text-sm transition-colors" title="Seleccionar Aleatorio">🎲</button>
                        <select id="ctrl_${i}" class="bg-gray-800 text-white p-1 rounded w-20 border border-gray-600 text-sm">
                            <option value="player" ${config.ctrl === 'player' ? 'selected' : ''}>Jugador</option>
                            <option value="bot" ${config.ctrl === 'bot' ? 'selected' : ''}>Bot</option>
                        </select>
                    </div>
                `;
            }
        }

        numBallsSelect.addEventListener('change', updatePlayerConfigs);
        document.getElementById('searchBall').addEventListener('input', updatePlayerConfigs);
        updatePlayerConfigs(); 

        function initTestModeSelect() {
            testBallTypeSelect.innerHTML = ALL_BALL_TYPES
                .filter(t => t.value !== 'random_pick')
                .map(t => `<option value="${t.value}">${t.label}</option>`)
                .join('') + `<option value="${TYPE_PRUEBA}">Pelota de Prueba (∞ HP)</option>`;
        }
        initTestModeSelect();

        btnEnterTest.addEventListener('click', () => {
            gameState = STATE_TEST;
            menu.classList.add('hidden');
            btnEnterTest.classList.add('hidden');
            testUI.classList.remove('hidden');
            clearGameElements();
            modifyingBallId = null;
            showMessage("Modo Prueba: Haz clic para añadir pelotas o bórralas con el modo eliminar", true);
        });

        btnExitTest.addEventListener('click', () => {
            gameState = STATE_MENU;
            testUI.classList.add('hidden');
            btnEnterTest.classList.remove('hidden');
            menu.classList.remove('hidden');
            modifyingBallId = null;
            clearGameElements();
            hideMessage();
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        });

        window.testModifyHP = (id, amount) => {
            const ball = balls.find(b => b.id === id);
            if (ball) ball.hp = Math.min(ball.maxHp, Math.max(0, ball.hp + amount));
        };

        window.testResetCD = (id) => {
            const ball = balls.find(b => b.id === id);
            if (!ball) return;
            ball.cooldown = 0; // General cooldown
          const timers = ['explosionCooldown', 'bombTimer', 'bombCooldown', 'ghostTimer', 'waterCooldown', 'iceCooldown', 'desertTimer', 'carnivoreCooldown', 'vampireAbilityCooldown', 'shieldCooldown', 'programDamageTimer', 'programHealTimer', 'deathAttackTimer', 'crystalAttractTimer', 'bichoAttackTimer', 'slideTimer', 'coinTossCooldown', 'dimensionalCooldown', 'rabbitKickTimer', 'rabbitCarrotTimer', 'wolfTimer', 'wolfCooldown', 'spaceTimer', 'spaceCooldown', 'lavaCooldown'];
            timers.forEach(t => { if (ball[t] !== undefined) ball[t] = 0; });
            floatingTexts.push(new FloatingText(ball.x, ball.y - 60, "¡RECARGADO!", '#3b82f6'));
        };

        window.toggleModifyBall = (id) => {
            if (modifyingBallId === id) {
                modifyingBallId = null;
                hideMessage();
            } else {
                modifyingBallId = id;
                const b = balls.find(ball => ball.id === id);
                deleteModeActive = false; 
                showMessage(`Editando: ${b.name}\n(Clics en mapa desactivados)`, true);
            }
            updateHUD();
        };

        function updateHUD() {
            if (gameState === STATE_MENU) {
                hud.classList.add('hidden');
                return;
            }
            hud.classList.remove('hidden');
            hudList.innerHTML = balls.filter(b => !b.isClone).map(b => {
                const healthPct = Math.max(0, (b.hp / b.maxHp) * 100);
                const isDead = !b.isAlive;
                const barColor = healthPct > 50 ? 'bg-green-500' : healthPct > 20 ? 'bg-orange-500' : 'bg-red-600';

                let cdVal = 0, cdMax = 1, hasCD = false, cdLabel = "READY";
                
                if (b.type === TYPE_ARMADA) { cdVal = b.cooldown; cdMax = 150; hasCD = true; cdLabel = "BALAS"; }
                else if (b.type === TYPE_RAPIDA) { cdVal = b.cooldown; cdMax = 210; hasCD = true; cdLabel = "DASH"; }
                else if (b.type === TYPE_VAMPIRO) { cdVal = b.vampireAbilityCooldown; cdMax = 258; hasCD = true; cdLabel = "VAMP"; }
                else if (b.type === TYPE_EXPLOSIVA) { cdVal = b.explosionCooldown; cdMax = 180; hasCD = true; cdLabel = "BOOM"; }
                else if (b.type === TYPE_BOMBA) { cdVal = b.bombActive ? b.bombTimer : b.bombCooldown; cdMax = b.bombActive ? 600 : 120; hasCD = true; cdLabel = b.bombActive ? "BOMBA" : "RECARGA"; }
                else if (b.type === TYPE_CERRADA) { cdVal = b.cooldown; cdMax = 520; hasCD = true; cdLabel = "JAULA"; }
                else if (b.type === TYPE_CLONADORA) { cdVal = b.cooldown; cdMax = 480; hasCD = true; cdLabel = "CLON"; }
                else if (b.type === TYPE_MAGNETICA) { cdVal = b.magnetTimer; cdMax = 180; hasCD = true; cdLabel = b.magnetPhase === 'attract' ? 'IMÁN+' : 'IMÁN-'; }
                else if (b.type === TYPE_FANTASMA) { 
                    hasCD = true;
                    if (b.isGhostMode) { cdVal = b.ghostTimer; cdMax = 300; cdLabel = "GHOST"; }
                    else { cdVal = b.cooldown; cdMax = 480; cdLabel = "WAIT"; }
                }
                else if (b.type === TYPE_ELECTRICA) { 
                    hasCD = true;
                    if (b.cooldown > 0) {
                        cdVal = b.cooldown; cdMax = 360; cdLabel = "RECARGA";
                    } else {
                        cdVal = b.charges < 20 ? (120 - b.chargeTimer) : 120;
                        cdMax = 120;
                        cdLabel = b.charges < 20 ? `⚡ ${b.charges}` : "⚡ MAX";
                    }
                }
                else if (b.type === TYPE_AGUA) { cdVal = b.waterCooldown; cdMax = 210; hasCD = true; cdLabel = "AGUA"; }
                else if (b.type === TYPE_FUEGO) { cdVal = b.cooldown; cdMax = 180; hasCD = true; cdLabel = "FUEGO"; }
                else if (b.type === TYPE_CARNIVORA) { cdVal = b.carnivoreCooldown; cdMax = 210; hasCD = true; cdLabel = "HAMBRE"; }
                else if (b.type === TYPE_PLANTADORA) { cdVal = b.cooldown; cdMax = 300; hasCD = true; cdLabel = "PLANTAR"; }
                else if (b.type === TYPE_SONORA) { cdVal = b.cooldown; cdMax = 240; hasCD = true; cdLabel = "SONIDO"; }
                else if (b.type === TYPE_GELATINA) { cdVal = b.cooldown; cdMax = 270; hasCD = true; cdLabel = "GEL"; }
                else if (b.type === TYPE_HIELO) { 
                    hasCD = true; 
                    if (b.iceAuraActive) { cdVal = b.iceAuraTimer; cdMax = 240; cdLabel = "AURORA"; }
                 
                    else { cdVal = b.iceCooldown; cdMax = 300; cdLabel = "HIELO"; } 
                }
                else if (b.type === TYPE_BOOMERANG) { cdVal = b.cooldown; cdMax = 300; hasCD = true; cdLabel = "BOOMERANG"; }
                else if (b.type === TYPE_ORBITAL) { cdVal = b.cooldown; cdMax = 330; hasCD = true; cdLabel = "PLANETA"; }
                else if (b.type === TYPE_CAMPANA) { hasCD = true; cdLabel = "MURO"; cdVal = 0; cdMax = 1; }
                else if (b.type === TYPE_PORTAL) { cdVal = b.portalCooldown; cdMax = 90; hasCD = true; cdLabel = "PORTAL"; }
                else if (b.type === TYPE_BICHO) { cdVal = b.cooldown; cdMax = 120; hasCD = true; cdLabel = "AGUIJÓN"; }
                else if (b.type === TYPE_DESERTICA) { 
                    hasCD = true; 
                    if (b.desertTimer > 0) { cdVal = b.desertTimer; cdMax = 360; cdLabel = "ARENA"; }
                    else { cdVal = b.cooldown; cdMax = 630; cdLabel = "ESPERA"; }
                }
                else if (b.type === TYPE_ESCUDERA) {
                    hasCD = true;
                    if (b.shieldHp > 0) { cdVal = b.shieldHp; cdMax = 20; cdLabel = "ESCUDO"; }
                    else { cdVal = b.shieldCooldown; cdMax = 750; cdLabel = "CARGA"; } 
                }
                else if (b.type === TYPE_MADERA) { cdVal = b.cooldown; cdMax = 180; hasCD = true; cdLabel = "MADERA"; }
                else if (b.type === TYPE_APOSTADORA) { cdVal = b.jackpotDamageTimer; cdMax = 360; hasCD = true; cdLabel = `🎰 ${Math.round(b.luck * 100)}%`; }
                else if (b.type === TYPE_DESLIZANTE) { 
                    if (b.slideTimer > 0) { cdVal = b.slideTimer; cdMax = 60; hasCD = true; cdLabel = "VELOZ"; }
                    else { cdVal = b.cooldown; cdMax = 240; hasCD = true; cdLabel = "REPOSO"; }
                }
                else if (b.type === TYPE_GRAVITACIONAL) {
                    hasCD = true;
                    if (b.gravityTimer > 0) { cdVal = b.gravityTimer; cdMax = 420; cdLabel = "GRAVEDAD"; }
                    else { cdVal = b.gravityCooldown; cdMax = 600; cdLabel = "ESPERA"; }
                }
                else if (b.type === TYPE_DIVISORA) {
                    hasCD = true;
                    if (b.divisoraTimer > 0) { cdVal = b.divisoraTimer; cdMax = 360; cdLabel = "DIVISIÓN"; }
                    else { cdVal = b.divisoraCooldown; cdMax = 600; cdLabel = "ESPERA"; }
                }
                else if (b.type === TYPE_TIEMPO) {
                    hasCD = true;
                    if (b.timeStopTimer > 0) { cdVal = b.timeStopTimer; cdMax = 120; cdLabel = "DETENIDO"; }
                    else { cdVal = b.cooldown; cdMax = 120; cdLabel = "RECARGA"; }
                }
                else if (b.type === TYPE_PIEDRA) {
                    hasCD = true;
                    if (b.isPetrified) { cdVal = b.stoneDuration; cdMax = 300; cdLabel = "PIEDRA"; }
                    else { cdVal = b.stoneTimer; cdMax = 420; cdLabel = "ESPERA"; }
                }
                else if (b.type === TYPE_CONEJO) { cdVal = b.rabbitCarrotTimer; cdMax = 720; hasCD = true; cdLabel = "ZANAHORIA"; }
                else if (b.type === TYPE_RAYO) {
                    hasCD = true;
                    cdVal = b.rayoTimer;
                    cdMax = b.rayoStrikeStage === 1 ? 60 : 210;
                    cdLabel = b.rayoStrikeStage === 1 ? "¡RAYO!" : "CARGA";
                }
                else if (b.type === TYPE_DIMENSIONAL) {
                    hasCD = true;
                    if (b.teleportEffectActive) { cdVal = b.teleportEffectTimer; cdMax = 300; cdLabel = "DESGARRO"; }
                    else { cdVal = b.dimensionalCooldown; cdMax = 420; cdLabel = "RECARGA"; }
                    
                }
                else if (b.type === TYPE_NUMERICA) {
                    hasCD = true;
                    if (b.numericTimer > 0) { cdVal = b.numericTimer; cdMax = 420; cdLabel = "ACTIVO"; }
                    else { cdVal = 480 - b.numericCooldown; cdMax = 480; cdLabel = "ESPERA"; }
                }
                else if (b.type === TYPE_PROYECCION) {
                    hasCD = true;
                    if (b.projectionTimer > 0) { cdVal = b.projectionTimer; cdMax = 90; cdLabel = "TRAZADO"; }
                    else { cdVal = 330 - b.projectionCooldown; cdMax = 330; cdLabel = "RECARGA"; }
                }
                else if (b.type === TYPE_DISCO) { cdVal = 330 - b.discoCooldown; cdMax = 330; hasCD = true; cdLabel = "DISCO"; }
                else if (b.type === TYPE_GOLEM) { cdVal = 210 - b.golemCooldown; cdMax = 210; hasCD = true; cdLabel = "GOLEM"; }
                else if (b.type === TYPE_DISCO) { cdVal = 360 - b.discoCooldown; cdMax = 360; hasCD = true; cdLabel = "DISCO"; }
                else if (b.type === TYPE_NUCLEAR) {
                    hasCD = true;
                    if (b.nuclearTimer > 0) { cdVal = 2700 - b.nuclearTimer; cdMax = 2700; cdLabel = "CARGA"; }
                    else { cdVal = 1; cdMax = 1; cdLabel = "MELTDOWN"; }
                }
                else if (b.type === TYPE_HOMBRELOBO) {
                    hasCD = true;
                    if (b.wolfTimer > 0) { cdVal = b.wolfTimer; cdMax = 360; cdLabel = "LOBO"; }
                    else { cdVal = 630 - b.wolfCooldown; cdMax = 630; cdLabel = "HUMANO"; }
                }
                else if (b.type === TYPE_ESPACIO) {
                    hasCD = true;
                    if (b.spaceTimer > 0) { cdVal = b.spaceTimer; cdMax = 780; cdLabel = "ESPACIO"; }
                    else { cdVal = 420 - b.spaceCooldown; cdMax = 420; cdLabel = "ESPERA"; }
                }
                else if (b.type === TYPE_CUESTIONADORA) {
                    hasCD = true;
                    if (b.questionTimer > 0) { cdVal = b.questionTimer; cdMax = 180; cdLabel = "CUESTIÓN"; }
                    else { cdVal = 420 - b.questionCooldown; cdMax = 420; cdLabel = "RECARGA"; }
                }
                else if (b.type === TYPE_ABSORVEDORA) {
                    hasCD = true;
                    cdVal = 6600 - b.absorbTimer; cdMax = 6600;
                    cdLabel = `ABSORB: ${Math.ceil(b.absorbedDamage)}`;
                }
                else if (b.type === TYPE_SOL) {
                    hasCD = true;
                    if (b.solUltActive) { cdVal = b.solUltTimer; cdMax = 180; cdLabel = "SUPERNOVA"; }
                    else { cdVal = b.solLevel; cdMax = 5; cdLabel = `SOL LVL ${b.solLevel}`; }
                }
                else if (b.type === TYPE_ESPIRITUAL) {
                    hasCD = true;
                    if (b.spiritualTimer > 0) { cdVal = b.spiritualTimer; cdMax = 4000; cdLabel = "TIEMPO"; }
                    else { cdVal = b.spiritualAttackTimer; cdMax = 24; cdLabel = "DESESPERO"; }
                }
                else if (b.type === TYPE_LAVA) { cdVal = 180 - b.lavaCooldown; cdMax = 180; hasCD = true; cdLabel = "VOLCÁN"; }
                else if (b.type === TYPE_METALICA) { cdVal = b.cooldown; cdMax = 240; hasCD = true; cdLabel = "METAL"; }
                else if (b.type === TYPE_CANON) { cdVal = b.cooldown; cdMax = 480; hasCD = true; cdLabel = "CAÑÓN"; }
                else if (b.type === TYPE_CRISTAL) { cdVal = b.crystalAttractTimer; cdMax = 600; hasCD = true; cdLabel = "ABSORVER"; }
                else if (b.type === TYPE_AMETRALLADORA) { 
                    hasCD = true;
                    cdVal = b.cooldown > 0 ? b.cooldown : b.machineGunCount;
                    cdMax = b.cooldown > 0 ? 840 : 15;
                    cdLabel = b.cooldown > 0 ? "RECARGA" : `MUNICIÓN ${b.machineGunCount}/15`;
                }
                else if (b.type === TYPE_MUERTE) {
                    hasCD = true;
                    cdVal = b.deathAttackTimer;
                    cdMax = 1800;
                    cdLabel = `💀 ${b.soulsCollected} ALMAS`;
                }
                else if (b.type === TYPE_FURIA) { hasCD = b.isFurious; cdVal = b.hp; cdMax = 40; cdLabel = "FURIA"; }
                else if (b.type === TYPE_ARANA) { hasCD = true; cdLabel = "ARÁCNIDA"; cdVal = 0; cdMax = 1; }
                else if (b.type === TYPE_PROGRAMADORA) {
                    hasCD = true;
                    cdVal = b.programDamageTimer;
                    cdMax = 240;
                    cdLabel = "CODE";
                }
                else if (b.type === TYPE_DRON) { cdVal = b.cooldown; cdMax = 240; hasCD = true; cdLabel = "DRON"; }
                else if (b.type === TYPE_TOPO) { cdVal = b.cooldown; cdMax = 420; hasCD = true; cdLabel = b.topoPhase > 0 ? "BAJO TIERRA" : "TOPO"; }
                else if (b.type === TYPE_PRUEBA) { hasCD = true; cdLabel = "INFINITY"; cdVal = 1; cdMax = 1; }

                const cdPct = hasCD ? (cdVal / cdMax) * 100 : 0;
                const cdBarColor = (b.type === TYPE_FANTASMA && b.isGhostMode) ? "bg-cyan-400" : "bg-blue-500";

                const isModifyingThis = modifyingBallId === b.id;
                const isModifyingOther = modifyingBallId !== null && modifyingBallId !== b.id;
                const isTest = gameState === STATE_TEST;

                let testControls = '';
                if (isTest && !isDead && !isModifyingOther) {
                    testControls = `
                        <div class="flex flex-col gap-1 mt-1 pointer-events-auto">
                        <div class="flex flex-col gap-1 mt-1">
                            <div class="flex gap-1">
                                <button onclick="testModifyHP(${b.id}, parseInt(document.getElementById('testHPStep').value))" class="bg-green-600 hover:bg-green-500 text-[9px] px-1.5 py-0.5 rounded text-white font-bold flex-1">+HP</button>
                                <button onclick="testModifyHP(${b.id}, -parseInt(document.getElementById('testHPStep').value))" class="bg-red-600 hover:bg-red-500 text-[9px] px-1.5 py-0.5 rounded text-white font-bold flex-1">-HP</button>
                            </div>
                            <button onclick="toggleModifyBall(${b.id})" class="${isModifyingThis ? 'bg-yellow-500' : 'bg-blue-500'} hover:opacity-80 text-[9px] px-1.5 py-0.5 rounded text-white font-bold w-full uppercase">
                                ${isModifyingThis ? 'Confirmar' : 'Modificar'}
                            </button>
                            ${!isModifyingThis ? `<button onclick="testResetCD(${b.id})" class="bg-gray-600 hover:bg-gray-500 text-[9px] px-1.5 py-0.5 rounded text-white font-bold w-full">RECARGAR</button>` : ''}
                        </div>
                    `;
                }

                return `
                    <div class="flex flex-col gap-1.5 ${isDead ? 'opacity-30 grayscale' : (isModifyingThis ? 'bg-yellow-500/20 ring-2 ring-yellow-500 rounded-lg p-2 shadow-lg shadow-yellow-500/20' : 'border-b border-gray-700/50 pb-2')} transition-all duration-300">
                        <div class="flex justify-between items-center gap-4">
                            <span class="font-bold text-sm" style="color: ${b.color}">${b.name} <span class="text-[10px] uppercase opacity-70">(${b.type})</span></span>
                            <span class="text-xs font-mono font-bold">${isDead ? 'ELIMINADO' : Math.ceil(b.hp) + ' HP'}</span>
                        </div>
                        <div class="w-full bg-gray-900 h-2.5 rounded-full overflow-hidden border border-gray-700">
                            <div class="h-full ${barColor} transition-all duration-300" style="width: ${healthPct}%"></div>
                        </div>
                        ${hasCD && !isDead ? `
                        <div class="flex items-center gap-2">
                            <span class="text-[9px] font-bold text-blue-300 w-10 uppercase">${cdLabel}</span>
                            <div class="flex-1 bg-gray-900 h-1.5 rounded-full overflow-hidden border border-gray-700/50">
                                <div class="h-full ${cdBarColor} transition-all duration-75" style="width: ${cdPct}%"></div>
                            </div>
                        </div>
                        ` : ''}
                        ${testControls}
                    </div>
                `;
            }).join('');
        }

        function clearGameElements() {
            projectiles = []; toxicZones = []; balls = []; floatingTexts = []; discs = [];
            modifyingBallId = null;
            explosionEffects = []; radioactiveTrails = []; closedSpaces = []; 
            waterPuddles = []; fireballs = []; vines = []; sonicWaves = []; 
            apples = []; boomerangs = []; orbitalPlanets = []; dongEffects = []; 
            cacti = []; laserBeamsV2 = []; woodenPlanks = []; crystalShards = []; darkSouls = []; spiderThreads = []; codeLines = []; ranking = [];
            rabbitKicks = []; carrots = [];
            rabbitKicks = []; carrots = []; sandiaSeeds = []; sandiaProjectiles = [];
            drones = [];
            meteorites = [];
            lavaVolcanoes = [];
            spiritualSouls = [];
        }

        function getEffectiveSpeed(baseSpeed) {
            return baseSpeed * gameSpeedMultiplier;
        }

        function showMessage(msg, keep = false) {
            messageOverlay.innerText = msg;
            messageOverlay.classList.remove('hidden');
            if (!keep) {
                setTimeout(() => messageOverlay.classList.add('hidden'), 2000);
            }
        }

        function hideMessage() {
            messageOverlay.classList.add('hidden');
        }

        btnStart.addEventListener('click', () => {
            const num = parseInt(numBallsSelect.value);
            clearGameElements(); 
            const margin = 50;
            const positions = [
                {x: margin, y: margin},
                {x: canvas.width - margin, y: canvas.height - margin},
                {x: canvas.width - margin, y: margin},
                {x: margin, y: canvas.height - margin}
            ];

            for (let i = 1; i <= num; i++) {
                let type = document.getElementById(`type_${i}`).value;
                const ctrl = document.getElementById(`ctrl_${i}`).value;
                const name = document.getElementById(`name_${i}`).value;

                if (type === 'random_pick') {
                    const pool = ALL_BALL_TYPES.filter(t => t.value !== 'random_pick');
                    type = pool[Math.floor(Math.random() * pool.length)].value;
                }

                if (type === TYPE_DOBLE) {
                    balls.push(new Ball(i, type, ctrl, positions[i-1].x - 25, positions[i-1].y, `${name}A`));
                    balls.push(new Ball(i, type, ctrl, positions[i-1].x + 25, positions[i-1].y, `${name}B`));
                } else {
                    balls.push(new Ball(i, type, ctrl, positions[i-1].x, positions[i-1].y, name));
                }
            }

            menu.classList.add('hidden');
            gameState = STATE_AIMING;
            btnToggleSpeed.classList.add('hidden'); 
            btnExitFight.classList.remove('hidden'); 
            processAiming();
        });

        btnRestart.addEventListener('click', () => {
            gameState = STATE_MENU;
            btnRestart.classList.add('hidden');
            btnExitFight.classList.add('hidden'); 
            btnToggleSpeed.classList.add('hidden'); 
            clearGameElements();
            hideMessage();
            menu.classList.remove('hidden');
            ctx.clearRect(0, 0, canvas.width, canvas.height); 
        });

        btnExitFight.addEventListener('click', () => {
            gameState = STATE_MENU;
            btnRestart.classList.add('hidden');
            btnExitFight.classList.add('hidden');
            btnToggleSpeed.classList.add('hidden'); 
            hideMessage();
            clearGameElements();
            menu.classList.remove('hidden');
            ctx.clearRect(0, 0, canvas.width, canvas.height); 
        });


        function processAiming() {
            balls.forEach(b => {
                if (b.controller === 'bot') {
                let angle = Math.random() * Math.PI * 2;
                b.vx = Math.cos(angle) * getEffectiveSpeed(BASE_SPEED);
                b.vy = Math.sin(angle) * getEffectiveSpeed(BASE_SPEED);
                }
            });
            showMessage("Apunta tus pelotas para comenzar", true);
        }

        canvas.addEventListener('mousedown', (e) => {
            if (gameState === STATE_TEST) {
                const rect = canvas.getBoundingClientRect();
                const mx = e.clientX - rect.left;
                const my = e.clientY - rect.top;

                if (modifyingBallId !== null) {
                    showMessage("Termina de modificar la pelota actual primero", false);
                    return;
                }

                if (deleteModeActive) {
                    const index = balls.findIndex(b => Math.hypot(b.x - mx, b.y - my) < b.radius);
                    if (index !== -1) {
                        balls.splice(index, 1);
                        floatingTexts.push(new FloatingText(mx, my, "ELIMINADA", '#ef4444'));
                    }
                } else {
                    const type = testBallTypeSelect.value;
                    const id = balls.length + 1;
                    if (type === TYPE_DOBLE) {
                        const b1 = new Ball(id, type, 'bot', mx - 25, my, `T${id}A`);
                        const b2 = new Ball(id, type, 'bot', mx + 25, my, `T${id}B`);
                        [b1, b2].forEach(nb => {
                            let angle = Math.random() * Math.PI * 2;
                            nb.vx = Math.cos(angle) * getEffectiveSpeed(BASE_SPEED);
                            nb.vy = Math.sin(angle) * getEffectiveSpeed(BASE_SPEED);
                            balls.push(nb);
                        });
                        floatingTexts.push(new FloatingText(mx, my, "¡DÚO AÑADIDO!", '#22c55e'));
                    } else {
                        const newBall = new Ball(id, type, 'bot', mx, my, `T${id}`);
                        let angle = Math.random() * Math.PI * 2;
                        newBall.vx = Math.cos(angle) * getEffectiveSpeed(BASE_SPEED);
                        newBall.vy = Math.sin(angle) * getEffectiveSpeed(BASE_SPEED);
                        balls.push(newBall);
                        floatingTexts.push(new FloatingText(mx, my, "¡AÑADIDA!", '#22c55e'));
                    }
                }
                return;
            }

            if (gameState === STATE_AIMING) {
                isDragging = true;
                const rect = canvas.getBoundingClientRect();
                const mx = e.clientX - rect.left;
                const my = e.clientY - rect.top;

                const ball = balls.find(b => b.isAlive && b.controller === 'player' && 
                    Math.hypot(b.x - mx, b.y - my) < b.radius && b.vx === 0);

                if (ball) {
                    isDragging = true;
                    selectedBall = ball;
                    dragStart = { x: mx, y: my };
                    dragCurrent = { ...dragStart };
                }
            }
        });

        canvas.addEventListener('mousemove', (e) => {
            if (isDragging) {
                const rect = canvas.getBoundingClientRect();
                dragCurrent = { x: e.clientX - rect.left, y: e.clientY - rect.top };
            }
        });

        canvas.addEventListener('mouseup', (e) => {
            if (isDragging && gameState === STATE_AIMING && selectedBall) {
                isDragging = false;
                let dx = dragCurrent.x - dragStart.x;
                let dy = dragCurrent.y - dragStart.y;
                
                if (Math.hypot(dx, dy) < 10) {
                    let angle = Math.random() * Math.PI * 2;
                    dx = Math.cos(angle); dy = Math.sin(angle);
                }

                let dist = Math.hypot(dx, dy);
                selectedBall.vx = (dx / dist) * getEffectiveSpeed(BASE_SPEED);
                selectedBall.vy = (dy / dist) * getEffectiveSpeed(BASE_SPEED);
                selectedBall = null;
            }
        });

        btnToggleDeleteMode.addEventListener('click', () => {
            deleteModeActive = !deleteModeActive;
            btnToggleDeleteMode.textContent = deleteModeActive ? "Modo Eliminar: ON" : "Modo Eliminar: OFF";
            btnToggleDeleteMode.classList.toggle('bg-red-600', deleteModeActive);
            btnToggleDeleteMode.classList.toggle('bg-gray-600', !deleteModeActive);
        });

        btnClearTest.addEventListener('click', () => {
            clearGameElements();
            floatingTexts.push(new FloatingText(canvas.width/2, canvas.height/2, "MAPA LIMPIO", '#ffffff'));
        });

        btnToggleSpeed.addEventListener('click', () => {
            if (gameSpeedMultiplier === 1) {
                gameSpeedMultiplier = 2;
                btnToggleSpeed.textContent = 'Velocidad x1';
            } else {
                gameSpeedMultiplier = 1;
                btnToggleSpeed.textContent = 'Velocidad x2';
            }
        });


        function resolveCollision(b1, b2, anyTimeStopped = false) {
            let dx = b2.x - b1.x;
            let dy = b2.y - b1.y;
            let distance = Math.hypot(dx, dy);

            if (distance === 0) return;
            if (b1.id === b2.id && b1.type !== TYPE_DOBLE) return; 
            
            if (distance === 0) return; 
            
             const skill1 = (b1.silenceTimer > 0) ? TYPE_NORMAL : ((b1.type === TYPE_ERROR && b1.currentErrorType) ? b1.currentErrorType : b1.type);
            const skill2 = (b2.type === TYPE_ERROR && b2.currentErrorType) ? b2.currentErrorType : b2.type;

            if ((skill1 === TYPE_FANTASMA && b1.isGhostMode) || (skill2 === TYPE_FANTASMA && b2.isGhostMode)) return;

            if ((skill1 === TYPE_VAMPIRO && b1.vampireAttachedTo === b2) || (skill2 === TYPE_VAMPIRO && b2.vampireAttachedTo === b1)) return;

            if ((skill1 === TYPE_CARNIVORA && b1.carnivoreTarget === b2) || (skill2 === TYPE_CARNIVORA && b2.carnivoreTarget === b1)) return;

            if (distance < b1.radius + b2.radius) {
                if (skill1 === TYPE_VAMPIRO && !b1.vampireAttachedTo && !b2.vampireAttachedTo && b1.vampireAbilityCooldown <= 0 && b1.grabbedTimer <= 0) {
                    b1.vampireAttachedTo = b2;
                    b1.vampireAttachTimer = 120; 
                    b1.vampireDrainCooldown = 30; 
                    return; 
                }
                if (skill2 === TYPE_VAMPIRO && !b2.vampireAttachedTo && !b1.vampireAttachedTo && b2.vampireAbilityCooldown <= 0 && b2.grabbedTimer <= 0) {
                    b2.vampireAttachedTo = b1;
                    b2.vampireAttachTimer = 120; 
                    b2.vampireDrainCooldown = 30; 
                    return; 
                }
            }

            if (distance < b1.radius + b2.radius) {
                let overlap = (b1.radius + b2.radius) - distance;
                let nx = dx / distance;
                let ny = dy / distance;
                
                b1.x -= nx * (overlap / 2);
                b1.y -= ny * (overlap / 2);
                b2.x += nx * (overlap / 2);
                b2.y += ny * (overlap / 2);

                let v1n = b1.vx * nx + b1.vy * ny;
                let v1t = b1.vx * -ny + b1.vy * nx;
                let v2n = b2.vx * nx + b2.vy * ny;
                let v2t = b2.vx * -ny + b2.vy * nx;

                if (v1n - v2n > 0) {
                    let v1nAfter = v2n;
                    let v2nAfter = v1n;
                    b1.vx = v1nAfter * nx - v1t * ny;
                    b1.vy = v1nAfter * ny + v1t * nx;
                    b2.vx = v2nAfter * nx - v2t * ny;
                    b2.vy = v2nAfter * ny + v2t * nx;
                }

                const isVampireAttaching = 
                    (skill1 === TYPE_VAMPIRO && b1.vampireAttachedTo === b2) ||
                    (skill2 === TYPE_VAMPIRO && b2.vampireAttachedTo === b1);
                
                if (skill1 === TYPE_TIEMPO && b1.timeStopTimer > 0) b2.takeDamage(6, b1);
                if (skill2 === TYPE_TIEMPO && b2.timeStopTimer > 0) b1.takeDamage(6, b2);

                if (skill1 === TYPE_NUMERICA && b1.numericTimer > 0) {
                    const col = Math.floor(b2.x / (canvas.width / 3));
                    const row = Math.floor(b2.y / (canvas.height / 3));
                    const idx = Math.min(8, Math.max(0, row * 3 + col));
                    const dmg = b1.numericGrid[idx] || 1;
                    b2.takeDamage(dmg, b1);
                }
                if (skill2 === TYPE_NUMERICA && b2.numericTimer > 0) {
                    const col = Math.floor(b1.x / (canvas.width / 3));
                    const row = Math.floor(b1.y / (canvas.height / 3));
                    const idx = Math.min(8, Math.max(0, row * 3 + col));
                    const dmg = b2.numericGrid[idx] || 1;
                    b1.takeDamage(dmg, b2);
                }
                
                // Lógica de Pelota Color (Colisión)
                if (skill1 === TYPE_COLOR && b1.grabbedTimer <= 0) {
                    b1.activeColors.forEach(c => {
                        if (c === 'rojo') b2.takeDamage(5, b1);
                        if (c === 'morado') { b2.silenceTimer = 360; floatingTexts.push(new FloatingText(b2.x, b2.y - 50, "¡SILENCIO!", '#a855f7')); }
                        if (c === 'azul') { b2.slowTimer = 360; b2.weaknessTimer = 360; floatingTexts.push(new FloatingText(b2.x, b2.y - 50, "¡DEBILITADO!", '#60a5fa')); }
                    });
                }

                if (skill2 === TYPE_COLOR && b2.grabbedTimer <= 0) {
                    b2.activeColors.forEach(c => {
                        if (c === 'rojo') b1.takeDamage(5, b2);
                        if (c === 'morado') { b1.silenceTimer = 360; floatingTexts.push(new FloatingText(b1.x, b1.y - 50, "¡SILENCIO!", '#a855f7')); }
                        if (c === 'azul') { b1.slowTimer = 360; b1.weaknessTimer = 360; floatingTexts.push(new FloatingText(b1.x, b1.y - 50, "¡DEBILITADO!", '#60a5fa')); }
                    });
                }


                if (skill1 === TYPE_PROYECCION && b1.projectionMoving) {
                    b2.takeDamage(4, b1);
                    b2.grabbedTimer = 120; // 2 segundos
                    b2.isFrozen = true;
                    floatingTexts.push(new FloatingText(b2.x, b2.y - 40, "¡CONGELADO!", '#818cf8'));
                }
                if (skill2 === TYPE_PROYECCION && b2.projectionMoving) {
                    b1.takeDamage(4, b2);
                    b1.grabbedTimer = 120; // 2 segundos
                    b1.isFrozen = true;
                    floatingTexts.push(new FloatingText(b1.x, b1.y - 40, "¡CONGELADO!", '#818cf8'));
                }

                if (!isVampireAttaching) {
                    if (skill1 === TYPE_NORMAL && b1.grabbedTimer <= 0) b2.takeDamage(5, b1);
                    if (skill2 === TYPE_NORMAL && b2.grabbedTimer <= 0) b1.takeDamage(5, b2);

                    if (skill1 === TYPE_FURIA && b1.grabbedTimer <= 0) {
                        if (b2.invulnTimer <= 0 && b1.isFurious) {
                            b1.hp = Math.min(b1.maxHp, b1.hp + 4);
                            floatingTexts.push(new FloatingText(b1.x, b1.y - 40, "+4 HP", '#22c55e'));
                        }
                        b2.takeDamage(b1.isFurious ? 5 : 2, b1);
                    }
                    if (skill2 === TYPE_FURIA && b2.grabbedTimer <= 0) {
                        if (b1.invulnTimer <= 0 && b2.isFurious) {
                            b2.hp = Math.min(b2.maxHp, b2.hp + 4);
                            floatingTexts.push(new FloatingText(b2.x, b2.y - 40, "+4 HP", '#22c55e'));
                        }
                        b1.takeDamage(b2.isFurious ? 5 : 2, b2);
                    }


                    if (skill1 === TYPE_ALEATORIA && b1.grabbedTimer <= 0) b2.takeDamage(b1.randomDamage, b1);
                    if (skill2 === TYPE_ALEATORIA && b2.grabbedTimer <= 0) b1.takeDamage(b2.randomDamage, b2);

                    if (skill1 === TYPE_HOMBRELOBO && b1.grabbedTimer <= 0) b2.takeDamage(b1.wolfTimer > 0 ? 8 : 2, b1);
                    if (skill2 === TYPE_HOMBRELOBO && b2.grabbedTimer <= 0) b1.takeDamage(b2.wolfTimer > 0 ? 8 : 2, b2);

                    if (skill1 === TYPE_RAPIDA && b1.isDashing && b1.grabbedTimer <= 0) b2.takeDamage(4, b1);
                    if (skill2 === TYPE_RAPIDA && b2.isDashing && b2.grabbedTimer <= 0) b1.takeDamage(4, b2);

                    if (skill1 === TYPE_MAGNETICA && b1.magnetPhase === 'attract' && b1.grabbedTimer <= 0) {
                        b2.takeDamage(5, b1);
                    }
                    if (skill2 === TYPE_MAGNETICA && b2.magnetPhase === 'attract' && b2.grabbedTimer <= 0) {
                        b1.takeDamage(5, b2);
                    }

                    if (skill1 === TYPE_GELATINA && b1.cooldown <= 0 && b2.isAlive && b2.grabbedTimer <= 0) {
                        const duration = 120 + Math.random() * 180; 
                        b2.grabbedTimer = duration;
                        const jellyTime = Math.floor(120 + Math.random() * 181); 
                        b2.grabbedTimer = jellyTime;
                        b2.isTrappedByJelly = true;
                        b2.takeDamage(2);
                        b1.cooldown = 210; 
                        floatingTexts.push(new FloatingText(b2.x, b2.y - 40, "¡PEGAJOSO!", '#f472b6'));
                    }
                    if (skill2 === TYPE_GELATINA && b2.cooldown <= 0 && b1.isAlive && b1.grabbedTimer <= 0) {
                       
                        const duration = 120 + Math.random() * 180; 
                        b1.grabbedTimer = duration;
                        const jellyTime = Math.floor(120 + Math.random() * 181);
                        b1.grabbedTimer = jellyTime;
                        b1.isTrappedByJelly = true;
                        b1.takeDamage(2);
                        b2.cooldown = 210;
                        floatingTexts.push(new FloatingText(b1.x, b1.y - 40, "¡PEGAJOSO!", '#f472b6'));
                    }

                    if (skill1 === TYPE_PLANTADORA && b2.grabbedTimer > 0 && b1.grabbedTimer <= 0) {
                        b2.takeDamage(4);
                    }
                    if (skill2 === TYPE_PLANTADORA && b1.grabbedTimer > 0 && b2.grabbedTimer <= 0) {
                        b1.takeDamage(4);
                    }

                    if (b1.id !== b2.id) {
                        if (b1.isClone && b1.grabbedTimer <= 0) b2.takeDamage(5);

                        if (b2.isClone && b2.grabbedTimer <= 0) b1.takeDamage(5);
                    }

                    if (skill1 === TYPE_APOSTADORA && b1.jackpotDamageTimer > 0) b2.takeDamage(10, b1);
                    if (skill2 === TYPE_APOSTADORA && b2.jackpotDamageTimer > 0) b1.takeDamage(10, b2);

                    if (skill1 === TYPE_DOBLE && b1.id !== b2.id && b1.grabbedTimer <= 0 && b1.deathTimer === 0) {
                        const now = Date.now();
                        const last = b2.lastHitByTeam[b1.id];
                        if (last && (now - last.time < 1000) && last.name !== b1.name) {
                            b2.takeDamage(6, b1);
                            b2.takeDamage(4, b1);
                            floatingTexts.push(new FloatingText(b2.x, b2.y - 40, "¡COMBO!", '#7dd3fc'));
                        } else { b2.takeDamage(2, b1); }
                        b2.lastHitByTeam[b1.id] = { time: now, name: b1.name };
                    }
                    if (skill2 === TYPE_DOBLE && b2.id !== b1.id && b2.grabbedTimer <= 0 && b2.deathTimer === 0) {
                        const now = Date.now();
                        const last = b1.lastHitByTeam[b2.id];
                        if (last && (now - last.time < 1000) && last.name !== b2.name) {
                            b1.takeDamage(6, b2);
                            b1.takeDamage(4, b2);
                            floatingTexts.push(new FloatingText(b1.x, b1.y - 40, "¡COMBO!", '#7dd3fc'));
                        } else { b1.takeDamage(2, b2); }
                        b1.lastHitByTeam[b2.id] = { time: now, name: b2.name };
                    }

                    if (skill1 === TYPE_DESLIZANTE && b1.slideTimer > 0) {
                        b2.takeDamage(b1.slideDamage, b1);
                        b1.slideTimer = 0;
                        b1.slideMultiplier = 1;
                        b1.slideDamage = 0;
                        b1.cooldown = 240; 
                    }
                    if (skill2 === TYPE_DESLIZANTE && b2.slideTimer > 0) {
                        b1.takeDamage(b2.slideDamage, b2);
                        b2.slideTimer = 0;
                        b2.slideMultiplier = 1;
                        b2.slideDamage = 0;
                        b2.cooldown = 240; 
                    }
                }

                if (skill1 === TYPE_ELECTRICA && b1.cooldown <= 0 && b1.charges > 0 && b1.grabbedTimer <= 0) {
                    b2.takeDamage(b1.charges * 2, b1);
                    b1.charges = 0;
                    b1.cooldown = 360; 
                }
                if (skill2 === TYPE_ELECTRICA && b2.cooldown <= 0 && b2.charges > 0 && b2.grabbedTimer <= 0) {
                    b1.takeDamage(b2.charges * 2, b2);
                    b2.charges = 0;
                    b2.cooldown = 360; 
                }
            }
        }
    
        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const anyTimeStopped = balls.some(b => {
                const bSkill = (b.type === TYPE_ERROR && b.currentErrorType) ? b.currentErrorType : b.type;
                return (bSkill === TYPE_TIEMPO && b.isAlive && b.timeStopTimer > 0) ||
                       (bSkill === TYPE_PROYECCION && b.isAlive && b.projectionTimer > 0);
            });
            if (anyTimeStopped) ctx.filter = 'grayscale(50%) brightness(80%)';

            ctx.strokeStyle = '#374151';
            ctx.lineWidth = 1;
            for (let i = 0; i <= canvas.width; i += 50) {
                ctx.beginPath(); ctx.moveTo(i, 0); ctx.lineTo(i, canvas.height); ctx.stroke();
                ctx.beginPath(); ctx.moveTo(0, i); ctx.lineTo(canvas.width, i); ctx.stroke();
            }

            ctx.filter = 'none'; 

            const isDesertActive = balls.some(b => b.type === TYPE_DESERTICA && b.isAlive && b.desertTimer > 0);
            if (isDesertActive) {
                ctx.save();
                ctx.fillStyle = 'rgba(194, 178, 128, 0.3)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.restore();
            }

            const isSpaceActive = balls.some(b => {
                const bSkill = (b.type === TYPE_ERROR && b.currentErrorType) ? b.currentErrorType : b.type;
                return bSkill === TYPE_ESPACIO && b.isAlive && b.spaceTimer > 0;
            });
            if (isSpaceActive) {
                ctx.save();
                ctx.fillStyle = 'rgba(15, 23, 42, 0.7)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = 'white';
                for(let i=0; i<15; i++) {
                    let x = (Math.sin(i*500) * 0.5 + 0.5) * canvas.width;
                    let y = (Math.cos(i*200) * 0.5 + 0.5) * canvas.height;
                    ctx.globalAlpha = 0.3 + Math.sin(Date.now()/500 + i) * 0.4;
                    ctx.fillRect(x, y, 2, 2);
                }
                ctx.restore();
            }

            const isNuclearMeltdown = balls.some(b => {
                const bSkill = (b.type === TYPE_ERROR && b.currentErrorType) ? b.currentErrorType : b.type;
                return bSkill === TYPE_NUCLEAR && b.isAlive && b.nuclearTimer <= 0;
            });
            if (isNuclearMeltdown) {
                ctx.save();
                ctx.fillStyle = 'rgba(34, 197, 94, 0.25)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.restore();
            }

            if (gameState === STATE_AIMING) {
                btnToggleSpeed.classList.add('hidden'); 
                balls.forEach(b => b.draw(ctx));
                
                if (isDragging && selectedBall) {
                    ctx.beginPath();
                    ctx.moveTo(selectedBall.x, selectedBall.y);
                    let dx = dragCurrent.x - dragStart.x;
                    let dy = dragCurrent.y - dragStart.y;
                    ctx.lineTo(selectedBall.x + dx, selectedBall.y + dy);
                    ctx.strokeStyle = 'rgba(255, 255, 255, 0.5)';
                    ctx.lineWidth = 3;
                    ctx.setLineDash([5, 5]);
                    ctx.stroke();
                    ctx.setLineDash([]);
                    ctx.closePath();
                    
                    ctx.beginPath();
                    ctx.arc(selectedBall.x + dx, selectedBall.y + dy, 5, 0, Math.PI*2);
                    ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
                    ctx.fill();
                    ctx.closePath();
                }

                balls.forEach(b => {
                    if (b.isAlive && b.controller === 'player' && b.vx === 0) {
                    ctx.beginPath();
                    ctx.arc(b.x, b.y, b.radius + 8 + Math.sin(Date.now() / 150) * 3, 0, Math.PI * 2);
                    ctx.strokeStyle = '#3b82f6';
                    ctx.lineWidth = 3;
                    ctx.stroke();
                    ctx.closePath();
                    }
                });

                if (balls.every(b => !b.isAlive || (b.vx !== 0 || b.vy !== 0))) {
                    gameState = STATE_PLAYING;
                    showMessage("¡A LUCHAR!");
                }
            }

            if (gameState === STATE_PLAYING || gameState === STATE_TEST) {
                if (gameState === STATE_PLAYING) btnToggleSpeed.classList.remove('hidden');
                
                if (anyTimeStopped) {
                    // No hacer nada extra globalmente
                }

                balls.forEach(b => b.update(balls));

                if (!anyTimeStopped) {
                    toxicZones.forEach(tz => tz.update(balls));
                    toxicZones = toxicZones.filter(tz => tz.life > 0);
                    waterPuddles.forEach(wp => wp.update(balls));
                    waterPuddles = waterPuddles.filter(wp => wp.life > 0);
                    vines.forEach(v => v.update(balls));
                    vines = vines.filter(v => v.life > 0);
                    fireballs.forEach(fb => fb.update(balls));
                    fireballs = fireballs.filter(fb => fb.active);
                    explosionEffects.forEach(ee => ee.update(balls));
                    explosionEffects = explosionEffects.filter(ee => ee.life > 0);
                    radioactiveTrails.forEach(rt => rt.update(balls));
                    radioactiveTrails = radioactiveTrails.filter(rt => rt.life > 0);
                    closedSpaces.forEach(cs => cs.update(balls));
                    closedSpaces = closedSpaces.filter(cs => cs.life > 0);
                    sonicWaves.forEach(sw => sw.update(balls));
                    sonicWaves = sonicWaves.filter(sw => sw.life > 0);
                    apples.forEach(a => a.update(balls));
                    apples = apples.filter(a => a.active && a.life > 0);
                    boomerangs.forEach(bm => bm.update(balls));
                    boomerangs = boomerangs.filter(bm => bm.active);
                    orbitalPlanets.forEach(p => p.update(balls));
                    orbitalPlanets = orbitalPlanets.filter(p => p.active);
                    dongEffects.forEach(de => de.update());
                    dongEffects = dongEffects.filter(de => de.active);
                    cacti.forEach(c => c.update(balls));
                    cacti = cacti.filter(c => c.active);
                    laserBeamsV2.forEach(lb => lb.update(balls));
                    laserBeamsV2 = laserBeamsV2.filter(lb => lb.life > 0);
                    woodenPlanks.forEach(wp => wp.update(balls));
                    woodenPlanks = woodenPlanks.filter(wp => wp.life > 0);
                    crystalShards.forEach(cs => cs.update(balls));
                    crystalShards = crystalShards.filter(cs => cs.active);
                    darkSouls.forEach(ds => ds.update(balls));
                    darkSouls = darkSouls.filter(ds => ds.active);
                    spiderThreads.forEach(st => st.update(balls));
                    spiderThreads = spiderThreads.filter(st => st.life > 0);
                    codeLines.forEach(cl => cl.update(balls));
                    codeLines = codeLines.filter(cl => cl.active);
                    drones.forEach(d => d.update(balls));
                    drones = drones.filter(d => d.active);
                    rabbitKicks.forEach(rk => rk.update(balls));
                    rabbitKicks = rabbitKicks.filter(rk => rk.active);
                    discs.forEach(d => d.update(balls));
                    sandiaSeeds.forEach(s => s.update(balls));
                    sandiaSeeds = sandiaSeeds.filter(s => s.active);
                    sandiaProjectiles.forEach(p => p.update(balls));
                    sandiaProjectiles = sandiaProjectiles.filter(p => p.active);
                    meteorites.forEach(m => m.update(balls));
                    meteorites = meteorites.filter(m => m.active);
                    lavaVolcanoes.forEach(lv => lv.update(balls));
                    blackBalls.forEach(bb => bb.update(balls));
                    lavaVolcanoes = lavaVolcanoes.filter(lv => lv.active);
                    spiritualSouls.forEach(ss => ss.update(balls));
                    spiritualSouls = spiritualSouls.filter(ss => ss.active);
                    discs = discs.filter(d => d.active);

                    carrots.forEach(c => c.update(balls));
                    carrots = carrots.filter(c => c.active);
                }

                toxicZones.forEach(tz => tz.draw(ctx));
                waterPuddles.forEach(wp => wp.draw(ctx));
                vines.forEach(v => v.draw(ctx));
                fireballs.forEach(fb => fb.draw(ctx));
                explosionEffects.forEach(ee => ee.draw(ctx));
                radioactiveTrails.forEach(rt => rt.draw(ctx));
                closedSpaces.forEach(cs => cs.draw(ctx));
                sonicWaves.forEach(sw => sw.draw(ctx));
                apples.forEach(a => a.draw(ctx));
                boomerangs.forEach(bm => bm.draw(ctx));
                orbitalPlanets.forEach(p => p.draw(ctx));
                dongEffects.forEach(de => de.draw(ctx));
                cacti.forEach(c => c.draw(ctx));
                laserBeamsV2.forEach(lb => lb.draw(ctx));
                woodenPlanks.forEach(wp => wp.draw(ctx));
                crystalShards.forEach(cs => cs.draw(ctx));
                darkSouls.forEach(ds => ds.draw(ctx));
                spiderThreads.forEach(st => st.draw(ctx));
                codeLines.forEach(cl => cl.draw(ctx));
                drones.forEach(d => d.draw(ctx));
                rabbitKicks.forEach(rk => rk.draw(ctx));
                carrots.forEach(c => c.draw(ctx));
                discs.forEach(d => d.draw(ctx));
                sandiaSeeds.forEach(s => s.draw(ctx));
                sandiaProjectiles.forEach(p => p.draw(ctx));
                meteorites.forEach(m => m.draw(ctx));
                lavaVolcanoes.forEach(lv => lv.draw(ctx));
                blackBalls.forEach(bb => bb.draw(ctx));
                spiritualSouls.forEach(ss => ss.draw(ctx));

                if (gameState !== STATE_TEST || Math.floor(Date.now() / 150) % 2 === 0) updateHUD();

                if (!anyTimeStopped) {
                    balls.forEach(b1 => {
                        if (b1.type === TYPE_ESPADACHINA && b1.isAlive && b1.swordHitCooldown <= 0 && b1.grabbedTimer <= 0) {
                        const swordX = b1.x + Math.cos(b1.swordAngle) * b1.swordOrbitRadius;
                        const swordY = b1.y + Math.sin(b1.swordAngle) * b1.swordOrbitRadius;
                        const swordTipX = swordX + Math.cos(b1.swordAngle + Math.PI / 2) * b1.swordLength;
                        const swordTipY = swordY + Math.sin(b1.swordAngle + Math.PI / 2) * b1.swordLength;
                        const swordHitRadius = 5; 

                        balls.forEach(b2 => {
                            if (b2.id !== b1.id && b2.isAlive) {
                                let dist = Math.hypot(swordTipX - b2.x, swordTipY - b2.y);
                                if (dist < swordHitRadius + b2.radius) {
                                    b2.takeDamage(6, b1);
                                    b1.swordRotationSpeed *= -1; 
                                    b1.swordHitCooldown = 30; 
                                }
                            }
                        });
                        }
                    });

                    for (let i = 0; i < balls.length; i++) {
                        for (let j = i + 1; j < balls.length; j++) {
                            if (balls[i].isAlive && balls[j].isAlive) {
                                resolveCollision(balls[i], balls[j], anyTimeStopped);
                            }
                        }
                    }

                    projectiles.forEach((p, index) => {
                        p.update();
                        if (p.active) {
                            balls.forEach(b => {
                                if (b.isAlive && b.id !== p.ownerId) {
                                    let dist = Math.hypot(p.x - b.x, p.y - b.y);
                                    if (dist < p.radius + b.radius) {
                                        let owner = balls.find(ob => ob.id === p.ownerId);
                                        b.takeDamage(p.damage, owner, false, false, false, true); 
                                        p.active = false;
                                    }
                                }
                            });
                        }
                    });
                    projectiles = projectiles.filter(p => p.active);
                }

                projectiles.forEach(p => p.draw(ctx));

                balls.forEach(b => b.draw(ctx));

                floatingTexts.forEach(ft => { ft.update(); ft.draw(ctx); });
                floatingTexts = floatingTexts.filter(ft => ft.life > 0);

                if (gameState === STATE_PLAYING) {
                    let vivos = balls.filter(b => b.isAlive && !b.isClone);
                    if (vivos.length <= 1) {
                        gameState = STATE_GAMEOVER;
                        
                        let finalRanking = [...vivos, ...[...ranking].reverse()];
                        let rankingMsg = "🏆 RESULTADOS 🏆\n";
                        
                        finalRanking.forEach((b, i) => {
                            let puesto = "";
                            if (i === 0) puesto = "PRIMER LUGAR";
                            else if (i === 1) puesto = "SEGUNDO LUGAR";
                            else if (i === 2) puesto = "TERCER LUGAR";
                            else if (i === 3) puesto = "CUARTO LUGAR";
                            rankingMsg += `${puesto}: ${b.name}\n`;
                        });

                        if (vivos.length === 0) rankingMsg = "¡EMPATE!\n" + rankingMsg;

                        showMessage(rankingMsg, true);
                        btnRestart.classList.remove('hidden');
                        btnExitFight.classList.add('hidden');
                    }
                }
            }

            if (gameState === STATE_GAMEOVER) {
                btnToggleSpeed.classList.add('hidden'); 
                balls.forEach(b => b.draw(ctx));
                floatingTexts.forEach(ft => { ft.update(); ft.draw(ctx); });
                floatingTexts = floatingTexts.filter(ft => ft.life > 0);
            }
            requestAnimationFrame(gameLoop);
        }

        gameLoop();

    </script>
</body>
</html>           
                  
       
