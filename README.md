<!DOCTYPE html>

<html lang="es">



<head>

    <meta charset="UTF-8">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Generar M3U</title>

    <style>

        body {

            font-family: 'Arial', sans-serif;

            background-color: #000;

            color: #fff;

            margin: 0;

            padding: 0;

            text-align: center;

        }



        #generateButton {

            background-color: #3498db;

            border: none;

            padding: 10px 20px;

            font-size: 18px;

            color: #fff;

            cursor: pointer;

            transition: background-color 0.3s;

        }



        #generateButton:hover {

            background-color: #2980b9;

        }



        #progressBar {

            display: none;

            width: 100%;

            margin-top: 20px;

        }



        #downloadLink {

            display: none;

            color: #3498db;

            margin-top: 20px;

            text-decoration: none;

            font-size: 16px;

        }



        input[type="checkbox"] {

            margin-right: 10px;

            width: 20px;

            height: 20px;

            appearance: none;

            -webkit-appearance: none;

            -moz-appearance: none;

            border: 2px solid #333;

            border-radius: 4px;

            background-color: #fff;

            vertical-align: middle;

            position: relative;

        }



        input[type="checkbox"]:checked {

            background-color: #007bff;

            border-color: #007bff;

        }



        label {

            display: inline-block;

            vertical-align: middle;

            font-size: 16px;

        }



        #channelList {

            display: grid;

            grid-template-columns: repeat(3, 1fr);

            grid-gap: 10px;

            margin-top: 10px;

        }



        .modal {

            display: none;

            position: fixed;

            z-index: 1;

            left: 0;

            top: 0;

            width: 100%;

            height: 100%;

            overflow: auto;

            background-color: rgba(0, 0, 0, 0.7);

            color: #000;

            /* Cambia el color del texto a negro */

        }



        .modal-content {

            background-color: #fff;

            margin: 10% auto;

            padding: 20px;

            border: 1px solid #888;

            width: 80%;

            max-width: 500px;

        }



        .close {

            color: #aaa;

            float: right;

            font-size: 28px;

            font-weight: bold;

            cursor: pointer;

        }



        .close:hover {

            color: #000;

        }

    </style>

</head>



<body>

    <div>

        <h1>¡Bienvenidos a MovistarTV! -> con TVMIX @TokyoGhoulh</h1>

        <p>Genera tus listas M3U para OTT / TIVIMATE</p>

        <button id="generateButton">¡Pulsa para Generar!</button>

        <progress id="progressBar" max="100" value="0"></progress>

        <a id="downloadLink" download="gracias_tokyo.m3u">Descargar M3U</a>

        <div>

            <h2>Selecciona los canales de tu M3U:</h2>

            <h3>Por defecto, están todos los canales de Movistar seleccionados</h3>

        </div>

        <form id="categoryList">

            <!-- Las categorias de TVMIX se ponen aquí -->

        </form>

        <form id="channelList">

            <!-- Los canales de movistar aquí -->

        </form>

        <div id="myModal" class="modal">

            <div class="modal-content">

                <span class="close">&times;</span>

                <h3>Canales en esta categoría:</h3>

                <ul id="channelListModal">

                    <!-- Aquí se muestran los canales de las categorias de TVMIX -->

                </ul>

            </div>

        </div>

        <script>

            async function cargarCategorias() {

                const tvTxtResponse = await fetch("https://ia904508.us.archive.org/31/items/tv_20231015_20231015/tv.txt");

                const tvTxtData = await tvTxtResponse.json();

                const categoryListForm = document.getElementById('categoryList');

                const channelListModal = document.getElementById('channelListModal');

                const modal = document.getElementById('myModal');

                const closeBtn = document.querySelector('.close');

                tvTxtData.forEach(category => {

                    const checkbox = document.createElement('input');

                    checkbox.type = 'checkbox';

                    checkbox.name = 'categories';

                    checkbox.value = category.name;

                    checkbox.checked = false;

                    const label = document.createElement('label');

                    label.appendChild(checkbox);

                    label.appendChild(document.createTextNode(category.name));

                    checkbox.addEventListener('click', () => {

                        if (checkbox.checked) {

                            channelListModal.innerHTML = '';

                            category.samples.forEach(sample => {

                                const channelName = sample.name;

                                const listItem = document.createElement('li');

                                listItem.textContent = channelName;

                                channelListModal.appendChild(listItem);

                            });

                            modal.style.display = 'block';

                        }

                    });

                    categoryListForm.appendChild(label);

                });

                closeBtn.addEventListener('click', () => {

                    modal.style.display = 'none';

                });

            }

            async function cargarOpcionesCanales() {

                const response = await fetch("https://ottcache.dof6.com/movistarplus/webplayer/OTT/contents/epg?preOffset=0&postOffset=0&mdrm=true&tlsstream=true&demarcation=true&version=8");

                const datos = await response.json();

                const channelListForm = document.getElementById('channelList');

                datos.forEach(item => {

                    const {

                        CodCadenaTv: CadenaCodigo,

                        Nombre: CadenaNombre

                    } = item;

                    const checkbox = document.createElement('input');

                    checkbox.type = 'checkbox';

                    checkbox.name = 'channels';

                    checkbox.value = CadenaCodigo;

                    checkbox.checked = true;

                    const label = document.createElement('label');

                    label.appendChild(checkbox);

                    label.appendChild(document.createTextNode(CadenaNombre));

                    channelListForm.appendChild(label);

                });

            }

            async function ExtraerKid(uriCadena) {

                const mpdfile = await fetch(uriCadena);

                const mpdstr = await mpdfile.text();

                const lugar = mpdstr.indexOf("cenc:default_KID=");

                const inicio = lugar + 18;

                const fin = inicio + 36;

                return mpdstr.substring(inicio, fin).replace(/-/g, "");

            }



            function keyToHex(key) {

                try {

                    const decodedKey = atob(key);

                    return Array.from(decodedKey, byte => ('0' + byte.charCodeAt(0).toString(16)).slice(-2)).join('');

                } catch (error) {

                    console.error('Error decoding base64 key:', error);

                    return '';

                }

            }



            function hexToBase64(hex) {

                const bytes = [];

                for (let c = 0; c < hex.length; c += 2) {

                    bytes.push(parseInt(hex.substr(c, 2), 16));

                }

                return btoa(String.fromCharCode.apply(String, bytes)).replace(/=+$/, '');

            }

            document.getElementById('generateButton').addEventListener('click', async () => {

                const UrlCache = "https://ottcache.dof6.com/movistarplus/webplayer/OTT/contents/epg?preOffset=0&postOffset=0&mdrm=true&tlsstream=true&demarcation=true&version=8";

                try {

                    const response = await fetch(UrlCache);

                    const datos = await response.json();

                    const selectedCategories = Array.from(document.querySelectorAll('input[name="categories"]:checked')).map(input => input.value);

                    const selectedChannels = Array.from(document.querySelectorAll('input[name="channels"]:checked')).map(input => input.value);

                    const tvTxtResponse = await fetch("https://ia904508.us.archive.org/31/items/tv_20231015_20231015/tv.txt");

                    const tvTxtData = await tvTxtResponse.json();

                    const progressBar = document.getElementById('progressBar');

                    progressBar.style.display = 'block';

                    let DatosM3U = '';

                    for (const category of tvTxtData) {

                        if (selectedCategories.includes(category.name)) {

                            const categoryName = category.name;

                            for (const sample of category.samples) {

                                const {

                                    name: sampleName,

                                    uri: CadenaMPD,

                                    kid: CadenaKID,

                                    key: CadenaKEY,

                                    extension: CadenaExtension

                                } = sample;

                                if (!CadenaKID || !CadenaKEY) {

                                    DatosM3U += `#EXTINF:-1 tvg-logo="" group-title="${categoryName}" tvg-id="",${sampleName}\n`;

                                    DatosM3U += `${CadenaMPD}\n\n`;

                                } else {

                                    const keys = CadenaKEY.split(',');

                                    const kids = CadenaKID.split(',');

                                    if (keys.length === 1 && kids.length === 1) {

                                        const CadenaKEYBase64 = btoa(keys[0]);

                                        const CadenaKIDBase64 = kids[0];

                                        const hexadecimalKeyFromBase64 = keyToHex(CadenaKEYBase64);

                                        const hexadecimalKidFromBase64 = keyToHex(CadenaKIDBase64);

                                        if (hexadecimalKeyFromBase64 && hexadecimalKidFromBase64) {

                                            DatosM3U += `#EXTINF:-1 tvg-logo="" group-title="${categoryName}" tvg-id="",${sampleName}\n`;

                                            DatosM3U += `#KODIPROP:inputstream.adaptive.manifest_type=mpd\n`;

                                            DatosM3U += `#KODIPROP:inputstream.adaptive.license_key=https://vercel-php-clearkey-hex-base64-json.vercel.app/api/results.php?keyid=${hexadecimalKidFromBase64}&key=${hexadecimalKeyFromBase64}\n`;

                                            DatosM3U += `#KODIPROP:inputstream.adaptive.license_type=clearkey\n`;

                                            DatosM3U += `${CadenaMPD}\n\n`;

                                        }

                                    } else {

                                        const concatenatedKeys = keys.map((key, index) => `${keyToHex(btoa(key))}:${keyToHex(btoa(kids[index]))}`).join(',');

                                        DatosM3U += `#EXTINF:-1 tvg-logo="" group-title="${categoryName}" tvg-id="",${sampleName}\n`;

                                        DatosM3U += `#KODIPROP:inputstream.adaptive.manifest_type=${CadenaExtension}\n`;

                                        DatosM3U += `#KODIPROP:inputstream.adaptive.license_key=${concatenatedKeys}\n`;

                                        DatosM3U += `#KODIPROP:inputstream.adaptive.license_type=clearkey\n`;

                                        DatosM3U += `${CadenaMPD}\n\n`;

                                    }

                                }

                            }

                        }

                    }

                    for (const item of datos) {

                        const {

                            CodCadenaTv: CadenaCodigo,

                            Nombre: CadenaNombre,

                            PuntoReproduccion: CadenaMPD,

                            Logo: CadenaLogo

                        } = item;

                        const CadenaKIDHex = await ExtraerKid(CadenaMPD);

                        const CadenaKIDBase64 = hexToBase64(CadenaKIDHex);

                        if (selectedChannels.includes(CadenaCodigo)) {

                            let hasAddedChannel = false;

                            for (const category of tvTxtData) {

                                for (const sample of category.samples) {

                                    if (sample.kid === CadenaKIDBase64) {

                                        const CadenaKEY = sample.key;

                                        const hexadecimalKey = keyToHex(btoa(CadenaKEY));

                                        if (hexadecimalKey) {

                                            if (!hasAddedChannel) {

                                                DatosM3U += `#EXTINF:-1 tvg-logo="${CadenaLogo}" group-title="Movistar" tvg-id="${CadenaCodigo}",${CadenaNombre}\n`;

                                                DatosM3U += `#KODIPROP:inputstream.adaptive.manifest_type=mpd\n`;

                                                DatosM3U += `#KODIPROP:inputstream.adaptive.license_key=https://vercel-php-clearkey-hex-base64-json.vercel.app/api/results.php?keyid=${CadenaKIDHex}&key=${hexadecimalKey}\n`;

                                                DatosM3U += `#KODIPROP:inputstream.adaptive.license_type=clearkey\n`;

                                                DatosM3U += `${CadenaMPD}\n\n`;

                                                hasAddedChannel = true;

                                            }

                                        }

                                    }

                                }

                            }

                            const progressPercentage = (datos.indexOf(item) + 1) / datos.length * 100;

                            progressBar.value = progressPercentage;

                        }

                    }

                    const blob = new Blob([DatosM3U], {

                        type: 'text/plain'

                    });

                    const url = URL.createObjectURL(blob);

                    const downloadLink = document.getElementById('downloadLink');

                    downloadLink.href = url;

                    downloadLink.download = 'gracias_tokyo.m3u';

                    downloadLink.style.display = 'block';

                    progressBar.style.display = 'none';

                } catch (error) {

                    console.error("Error:", error);

                }

            });

            cargarCategorias();

            cargarOpcionesCanales();

        </script>

</body>



</html>

<!---
Reque88/Reque88 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
