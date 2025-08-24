<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Encuesta de Tabaquismo</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="min-h-screen bg-gray-50 text-gray-900">
  <main class="max-w-2xl mx-auto py-10 px-4">
    <header class="mb-8"
      <p class="text-sm text-gray-600 mt-2">Este cuestionario está diseñado para identificar pacientes con riesgo de cáncer de pulmón (según guías NCCN 2025). Las respuestas son anónimas y sólo se usarán con fines académicos.</p>
    </header>

    <form id="surveyForm" class="space-y-6 bg-white p-6 rounded-2xl shadow">
      <div>
        <label class="block text-sm font-medium">1. ¿Cuántos años tiene?</label>
        <input type="number" name="edad" min="0" max="120" class="mt-1 w-full rounded-xl border-gray-300 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500" required />
        <p class="text-xs text-gray-500">*Continuar sólo si es mayor a 50 años*</p>
      </div>

      <div>
        <label class="block text-sm font-medium">2. ¿Cuántos años lleva fumando o fumó?</label>
        <input type="number" name="anios_fumando" min="0" class="mt-1 w-full rounded-xl border-gray-300 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500" required />
        <p class="text-xs text-gray-500">*Si ≥ 20 años, es candidato para TC de baja dosis según NCCN 2025*</p>
      </div>

      <div>
        <label class="block text-sm font-medium">3. Durante ese tiempo, ¿cuántos cigarros fuma o fumó al día?</label>
        <input type="number" name="cigarros_dia" min="0" class="mt-1 w-full rounded-xl border-gray-300 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500" required />
      </div>

      <div>
        <label class="block text-sm font-medium">4. Índice tabáquico (IT)</label>
        <p class="text-xs text-gray-500 mb-2">El IT se calcula como:<br> IT = (Cigarros por día × Años fumando) / 20</p>
        <input id="indice_tabaco" type="text" readonly class="w-full rounded-xl border-gray-300 bg-gray-100 px-3 py-2" placeholder="Se calculará automáticamente" />
      </div>

      <div class="flex items-start gap-3">
        <input id="consent" type="checkbox" required class="mt-1" />
        <label for="consent" class="text-sm text-gray-700">Confirmo que he leído el aviso de privacidad y doy mi consentimiento para el uso académico de mis respuestas.</label>
      </div>

      <button type="submit" class="w-full py-3 rounded-2xl bg-indigo-600 hover:bg-indigo-700 text-white font-semibold shadow">Enviar respuestas</button>
      <p id="status" class="text-sm mt-3"></p>
    </form>
  </main>

  <script>
    // ======== CONFIGURACIÓN DE SUPABASE ========
    const SUPABASE_URL = "https://TU-PROYECTO.supabase.co"; // reemplaza con tu URL
    const SUPABASE_ANON_KEY = "TU-ANON-KEY"; // reemplaza con tu anon key
    const TABLE = "respuestas";

    async function saveToSupabase(payload) {
      const resp = await fetch(`${SUPABASE_URL}/rest/v1/${TABLE}`, {
        method: 'POST',
        headers: {
          'apikey': SUPABASE_ANON_KEY,
          'Authorization': `Bearer ${SUPABASE_ANON_KEY}`,
          'Content-Type': 'application/json',
          'Prefer': 'return=representation'
        },
        body: JSON.stringify(payload)
      });
      if (!resp.ok) {
        const txt = await resp.text();
        throw new Error(`Error ${resp.status}: ${txt}`);
      }
      return await resp.json();
    }

    // ======== LÓGICA DE LA ENCUESTA ========
    function calcularIT(){
      const anios = Number(document.querySelector('[name="anios_fumando"]').value)||0;
      const cigarros = Number(document.querySelector('[name="cigarros_dia"]').value)||0;
      const it = (cigarros * anios)/20;
      document.getElementById('indice_tabaco').value = it ? it.toFixed(1) : '';
      return it;
    }

    document.querySelector('[name="anios_fumando"]').addEventListener('input', calcularIT);
    document.querySelector('[name="cigarros_dia"]').addEventListener('input', calcularIT);

    const form = document.getElementById('surveyForm');
    const statusEl = document.getElementById('status');

    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      statusEl.textContent = 'Enviando…';
      statusEl.className = 'text-sm mt-3 text-gray-600';

      try {
        const edad = Number(document.querySelector('[name="edad"]').value);
        const anios = Number(document.querySelector('[name="anios_fumando"]').value);
        const cigarros = Number(document.querySelector('[name="cigarros_dia"]').value);
        const it = calcularIT();
        const consentimiento = document.getElementById('consent').checked;

        const payload = {
          edad,
          anios_fumando: anios,
          cigarros_dia: cigarros,
          indice_tabaco: it,
          consentimiento,
          created_at: new Date().toISOString()
        };

        if (!consentimiento) {
          statusEl.textContent = 'Debes aceptar el consentimiento.';
          statusEl.className = 'text-sm mt-3 text-red-600';
          return;
        }

        await saveToSupabase(payload);
        statusEl.textContent = '¡Gracias! Tus respuestas se registraron correctamente.';
        statusEl.className = 'text-sm mt-3 text-green-600';
        form.reset();
        document.getElementById('indice_tabaco').value = '';
      } catch (err) {
        console.error(err);
        statusEl.textContent = 'Hubo un problema al guardar. Intenta de nuevo o contacta al responsable.';
        statusEl.className = 'text-sm mt-3 text-red-600';
      }
    });
  </script>
</body>
</html>
