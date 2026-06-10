% =====================================================================
%   SISTEMA ECO-INTELIGENTE DE OPTIMIZACIÓN DE RIEGO (Versión Interactiva)
%   Materia: Mecatrónica / Ingeniería
% =====================================================================
clear; clc; close all;

fprintf('====================================================\n');
fprintf('   SISTEMA ECO-INTELIGENTE DE OPTIMIZACIÓN DE RIEGO  \n');
fprintf('====================================================\n\n');

fprintf('⚠️ NOTA: Si tardas mucho en escribir un dato en la consola,\n');
fprintf('el sistema asignará un valor de respaldo automáticamente.\n\n');

% ---------------------------------------------------------------------
% 1. ENTRADA INTERACTIVA DE DATOS (Con sistema de respaldo ante Timeouts)
% ---------------------------------------------------------------------
fprintf('--- CONFIGURACIÓN DE SENSORES ---\n');

% Entrada para Tipo de Suelo
tipo_suelo = input('Tipo de suelo (1=Arenoso, 2=Franco, 3=Arcilloso) [Defecto=2]: ');
if isempty(tipo_suelo)
    tipo_suelo = 2; % Respaldo si das enter vacío o hay timeout
end

% Entrada para Humedad
humedad = input('Humedad actual medida en el suelo (%%) [Defecto=12.5]: ');
if isempty(humedad)
    humedad = 12.5; 
end

% Entrada para Temperatura
temperatura = input('Temperatura ambiente actual (°C) [Defecto=32.0]: ');
if isempty(temperatura)
    temperatura = 32.0; 
end

% Entrada para Pronóstico Climático
fprintf('Pronóstico del clima actual (Ej: "Lluvia fuerte", "Soleado")\n');
pronostico = input('Pronóstico [Defecto="Tormenta con lluvia fuerte"]: ', 's');
if isempty(pronostico)
    pronostico = 'Tormenta electrica con lluvia fuerte en 1 hora'; 
end

% ---------------------------------------------------------------------
% 2. LÓGICA DE INGENIERÍA: FÍSICA DE SUELOS 
% ---------------------------------------------------------------------
umbrales = [10, 20;   % 1: Arenoso
            18, 30;   % 2: Franco
            25, 42];  % 3: Arcilloso

nombres_suelo = {'Arenoso', 'Franco', 'Arcilloso'};
if tipo_suelo < 1 || tipo_suelo > 3
    tipo_suelo = 2; 
end
suelo_actual = nombres_suelo{tipo_suelo};

limite_critico = umbrales(tipo_suelo, 1);
limite_optimo = umbrales(tipo_suelo, 2);

estado_fisico = 'OPTIMO';
diagnostico = 'El suelo cuenta con reservas de agua suficientes.';
necesita_ia = 0;

if humedad < limite_critico
    estado_fisico = 'CRITICO';
    diagnostico = sprintf('Humedad por debajo del Punto de Marchitez (%d%%).', limite_critico);
    necesita_ia = 1;
elseif humedad < limite_optimo
    estado_fisico = 'ALERTA';
    diagnostico = sprintf('Humedad por debajo de la Capacidad de Campo optima (%d%%).', limite_optimo);
    necesita_ia = 1;
end

% Despliegue de Resultados Clínicos en Consola
fprintf('\n--- PROCESANDO LOGICA DE INGENIERIA ---\n');
fprintf('Suelo Evaluado: %s\n', suelo_actual);
fprintf('Humedad Registrada: %.1f%%\n', humedad);
fprintf('Temperatura Ambiente: %.1f°C\n', temperatura);
fprintf('Pronóstico Climático: %s\n', pronostico);
fprintf('>> Resultado Algoritmo: %s\n', estado_fisico);
fprintf('>> Diagnóstico: %s\n\n', diagnostico);

% ---------------------------------------------------------------------
% 3. GENERACIÓN DE GRÁFICA DE INGENIERÍA
% ---------------------------------------------------------------------
figure(1);
hold on;

% Dibujar fondos de riesgo
bar(1, limite_optimo * 1.5, 'FaceColor', [0.4, 0.8, 0.4], 'EdgeColor', 'none'); 
bar(1, limite_optimo, 'FaceColor', [0.9, 0.9, 0.4], 'EdgeColor', 'none');       
bar(1, limite_critico, 'FaceColor', [0.9, 0.4, 0.4], 'EdgeColor', 'none');      

% Dibujar barra del sensor
bar(1, humedad, 0.3, 'FaceColor', [0.1, 0.3, 0.8], 'EdgeColor', [0,0,0], 'LineWidth', 1.5);

% Líneas de límites
line([0.5, 1.5], [limite_critico, limite_critico], 'Color', [0.5, 0, 0], 'LineWidth', 2, 'LineStyle', '--');
line([0.5, 1.5], [limite_optimo, limite_optimo], 'Color', [0, 0.5, 0], 'LineWidth', 2, 'LineStyle', '--');

xlim([0.5, 1.5]);
ylim([0, limite_optimo * 1.4]);
set(gca, 'XTick', 1, 'XTickLabel', {['Suelo ' suelo_actual]});
ylabel('Humedad del Suelo (%)', 'FontWeight', 'bold');
title(['Analisis de Humedad - Estado: ' estado_fisico], 'FontWeight', 'bold');

text(0.52, limite_critico - 1.5, 'Pto. Marchitez (Critico)', 'Color', [0.5, 0, 0], 'FontSize', 9);
text(0.52, limite_optimo - 1.5, 'Cap. Campo (Optimo)', 'Color', [0, 0.5, 0], 'FontSize', 9);
text(1.2, humedad, [num2str(humedad) '%'], 'FontWeight', 'bold', 'Color', [0.1, 0.3, 0.8]);

grid on;
hold off;

% ---------------------------------------------------------------------
% 4. GENERACIÓN DE PAYLOAD PARA GEMINI IA
% ---------------------------------------------------------------------
if necesita_ia == 1
    fprintf('====================================================\n');
    fprintf('   PAYLOAD LISTO PARA COPIAR A GEMINI (PROMPT)      \n');
    fprintf('====================================================\n');
    fprintf('Actuas como el modulo de control inteligente del sistema de riego.\n');
    fprintf('Datos procesados en Octave: Suelo %s, Humedad %.1f%%, Temp %.1fC.\n', suelo_actual, humedad, temperatura);
    fprintf('Contexto de Pronostico: %s\n', pronostico);
    fprintf('Instruccion: Determina si aplica MANTENER RIEGO o BYPASS ACTIVADO. Devuelve formato estricto: DECISION FINAL y JUSTIFICACION TECNICA.\n');
end
