# Simulador de Atenuación de Rayos X

![Versión](https://img.shields.io/badge/versión-Beta%200.1-d4ff3a)
![Stack](https://img.shields.io/badge/stack-HTML%20%2B%20Three.js-orange)
![Licencia](https://img.shields.io/badge/licencia-MIT-blue)
![Status](https://img.shields.io/badge/status-en%20desarrollo-yellow)

Simulador interactivo desarrollado para el **Laboratorio Práctico N°1 — Atenuación por Aluminio y Cobre en Radiología** del curso *Protección Radiológica y Radiobiología I* del Departamento de Tecnología Médica, Universidad de Chile.

Reproduce numéricamente las mediciones de kerma en aire que se obtienen en sala con un equipo radiológico (45–150 kVp), una cámara de ionización tipo Farmer **NE 2571** o un detector de estado sólido **Radcal AGMS-D**, y láminas de Al/Cu de distintos espesores. Todo el modelo físico (espectro de Kramers, filtración inherente, atenuación NIST, eficiencia del detector y retrodispersión) corre íntegramente en el navegador, sin servidor.

> ⚠️ Versión **Beta 0.1**. Pensado como herramienta docente y de apoyo al práctico, **no como sustituto de mediciones reales** en contextos clínicos o regulatorios.
>
> ## Acceso en línea
>
> 👉 **https://lucianotejadac.github.io/simulador-atenuacion-rx/**
>
> No requiere instalación. Basta con un navegador moderno con WebGL (Chrome ≥ 90, Firefox ≥ 88, Edge ≥ 90, Safari ≥ 14).
>
> ---
>
> ## Contexto académico
>
> El práctico tiene por objetivo determinar el primer y segundo espesor hemirreductor (**HVL₁** y **HVL₂**), el coeficiente de homogeneidad (**Iₕ**) y la **energía efectiva** de un haz de rayos X de radiología, mediante el ajuste de la curva semi-logarítmica de atenuación.
>
> Marco teórico utilizado por el práctico:
>
> $$I(x) = I_0\, e^{-\mu x} = I_0\cdot 2^{-x/HVL}$$
>
> $$HVL = \frac{\ln 2}{\mu}$$
>
> Para haces polienergéticos con geometría de haz ancho la atenuación se aproxima por tramos:
>
> $$I_H = \frac{HVL_1}{HVL_2}$$
>
> Para haces monoenergéticos y geometría de haz estrecho, $I_H = 1$.
>
> ---
>
> ## Características
>
> - **Escena 3D interactiva** (WebGL/Three.js) con sala de RX, mesa radiológica DRGEM, columna, brazo del tubo, colimador, haz cónico y soporte de plumavit con textura procedural.
> - - **Dos detectores modelados con su geometría real**:
>   -   - Cámara Farmer **NE 2571**: cilindro con conector, anillos, vástago de aluminio, dedal transparente y cable coaxial azul.
>       -   - Sólido **AGMS-D**: puck rectangular negro con bisel, anillo metálico alrededor del área activa de 10 × 10 mm² y cable lateral.
>           - - **Controles tipo consola industrial** (panel izquierdo): energía (kVp), corriente·tiempo (mAs), DFD, altura del detector, colimación, material y espesor del atenuador.
>             - - **Steppers ± para los parámetros principales**:
>               -   - kVp en pasos de 5 (45 → 150)
>                   -   - mAs en serie radiológica R'10 (1, 1.25, 1.6, 2, 2.5, 3.2, 4, 5, 6.3, 8, 10, 12.5, 16, 20, 25, 32, 40, 50, 63, 80, 100)
>                       -   - Espesor: 0.5 mm en Al (0–10 mm) o 0.1 mm en Cu (0–2 mm)
>                           - - **Bloqueo automático tras la primera exposición**: sólo el espesor del atenuador puede modificarse hasta presionar LIMPIAR. Reproduce la disciplina experimental: una vez fijada la geometría y el detector, sólo se varía el filtro.
>                             - - **Límite de 30 disparos** por sesión, con contador y reset manual.
>                               - - **Registro tabular** de mediciones (#, material, espesor, kerma en μGy).
>                                 - - **Ruido estadístico gaussiano** en cada lectura ($\sigma \sim \sqrt{N}$ + piso electrónico 0.5 %).
>                                  
>                                   - ---
>
> ## Modelo físico
>
> Toda la implementación, las constantes y la documentación inline están embebidas en el archivo HTML único. La justificación detallada y las tablas NIST están en el bloque comentado `<!-- DOCUMENTACIÓN TÉCNICA -->` al inicio del `<body>`.
>
> ### Espectro polienergético (Kramers, 4 componentes)
>
> $$N(E) \propto Z\cdot\left(\frac{E_{max}}{E} - 1\right)$$
>
> Muestreado en cuatro energías representativas con $f_i \in \{0.15, 0.33, 0.55, 0.75\}$ y pesos crudos $w_i = (1/f_i - 1)$ que se renormalizan tras aplicar la filtración inherente.
>
> ### Filtración inherente del tubo
>
> Valor fijo de **4.605 mm Al equivalente**, que pre-endurece el espectro antes de cualquier atenuador externo. Incluye contribuciones físicas de: ventana de Berilio, aceite dieléctrico de refrigeración y espejo del colimador.
>
> ### Atenuación primaria (Beer–Lambert)
>
> Coeficientes de atenuación másica $\mu/\rho$ tomados de tablas **NIST** e interpolados log-log entre 10 y 150 keV.
>
> | Material | ρ (g/cm³) | Rango tabulado |
> |----------|-----------|----------------|
> | Aluminio | 2.699 | 10–150 keV |
> | Cobre    | 8.96  | 10–150 keV |
>
> ### Eficiencia energética del detector
>
> | Detector | Material activo | ρ (g/cm³) | L | Tabla μ_en/ρ |
> |----------|-----------------|-----------|---|--------------|
> | NE 2571 (Farmer) | aire seco | 1.205 × 10⁻³ | 24 mm | NIST aire |
> | AGMS-D (estado sólido) | silicio | 2.33 | 0.5 mm | NIST Si |
>
> ### Output factor del tubo (recalibrado 2026-04)
>
> | Detector | μGy/mAs a 70 kVp, 100 cm |
> |----------|--------------------------|
> | Cámara de ionización NE 2571 | **44.0** |
> | Estado sólido AGMS-D | **28.0** |
>
> ---
>
> ## Calibración y validación
>
> Los output_factor fueron recalibrados en abril de 2026 contra **cuatro datasets experimentales** obtenidos durante el práctico. Error medio **< 5 %** en el régimen relevante (cámara a 70 kVp con filtros hasta 6 mm Al o 0.7 mm Cu).
>
> ### Curva de Al · cámara · 70 kVp (dataset C)
>
> | Espesor | T modelo | T medido | Δ |
> |---------|----------|----------|---|
> | 0.5 mm | 0.855 | 0.850 | +0.6 % |
> | 1.0 mm | 0.742 | 0.744 | −0.3 % |
> | 2.0 mm | 0.576 | 0.583 | −1.2 % |
> | 4.0 mm | 0.383 | 0.386 | −0.8 % |
> | 6.0 mm | 0.275 | 0.270 | +1.8 % |
>
> ### Curva de Cu · cámara · 70 kVp
>
> | Espesor | T modelo | T medido | Δ |
> |---------|----------|----------|---|
> | 0.1 mm | 0.470 | 0.496 | −5.2 % |
> | 0.2 mm | 0.300 | 0.292 | +2.7 % |
> | 0.3 mm | 0.202 | 0.199 | +1.5 % |
> | 0.5 mm | 0.103 | 0.097 | +6.4 % |
> | 0.7 mm | 0.057 | 0.059 | −3.3 % |
>
> ---
>
> ## Uso
>
> ### Flujo del práctico
>
> 1. Confirmar **kVp** (default 70) y **mAs** (default 10) con los botones ±.
> 2. 2. Ajustar **DFD**, **altura del detector** sobre el plumavit y **colimación** del haz.
>    3. 3. Elegir el **detector** (NE 2571 o AGMS-D). La escena 3D cambia su geometría.
>       4. 4. Tomar la lectura **sin atenuador** (I₀): clic en EXPONER.
>          5. 5. Tras la primera exposición, todos los controles quedan **bloqueados** salvo el espesor del atenuador.
>             6. 6. Seleccionar Al o Cu, ir incrementando el espesor con los botones ±, y exponer en cada espesor.
>                7. 7. La columna kerma (μGy) se llena con cada disparo. Hasta **30 disparos** por sesión.
>                   8. 8. LIMPIAR para resetear contador y desbloquear los controles.
>                     
>                      9. > Los HVL₁, HVL₂, Iₕ y la energía efectiva se calculan **fuera del simulador**, en el informe de el/la estudiante, a partir de la tabla de mediciones (escala semi-log y ecuaciones del práctico).
>                         >
>                         > ---
>                         >
>                         > ## Stack técnico
>                         >
>                         > | Capa | Tecnología |
>                         > |------|------------|
>                         > | UI | HTML5 + CSS3 (grid + custom properties) |
>                         > | Lógica | JavaScript ES6 (vanilla, sin transpilación) |
>                         > | 3D | [Three.js r128](https://threejs.org) (CDN, sin bundler) |
>                         > | Tipografías | Fraunces + JetBrains Mono, vía Google Fonts |
>                         > | Datos físicos | Tablas NIST embebidas (μ/ρ y μ_en/ρ para Al, Cu, aire, Si entre 10 y 150 keV) |
>                         >
>                         > Sin dependencias de build, sin npm, sin servidor. El HTML es **autocontenido**.
>                         >
>                         > ---
>                         >
>                         > ## Limitaciones conocidas
>                         >
>                         > - **Espectro Kramers truncado en f = 0.75 · kVp**. A 81 kVp con filtros gruesos (≥ 4 mm Al o ≥ 0.3 mm Cu) el modelo subestima la transmisión hasta un 25 %. Para el rango clínico de 70 kVp del práctico el error es despreciable.
>                         > - - **Geometría discreta en la UI**: kVp en pasos de 5 (no permite 81 kVp exacto), mAs en serie R'10 (no incluye 14 ni 22).
>                         >   - - **Sin endurecimiento adicional dependiente del espesor del atenuador externo**: la dependencia $HVL_1 \neq HVL_2$ surge del espectro polienergético, pero no hay un loop iterativo de hardening secundario.
>                         >     - - **Sin dispersión coherente ni transporte de fluencia**: sólo absorción primaria y un factor empírico de retrodispersión desde la mesa.
>                         >       - - **k_TP no se aplica a la lectura**: el simulador devuelve directamente kerma en aire físico.
>                         >        
>                         >         - ---
>                         >
>                         > ## Roadmap
>                         >
>                         > - [ ] Quinta componente Kramers en f = 0.92 para mejorar el ajuste a 81 kVp con filtros gruesos.
>                         > - [ ] - [ ] Exportación CSV/JSON del registro de mediciones.
>                         > - [ ] - [ ] Modo "tutor" con pistas paso a paso del flujo del práctico.
>                         > - [ ] - [ ] Soporte multi-kVp en la misma sesión (con varios I₀ de referencia).
>                         > - [ ] - [ ] Empaquetado totalmente offline (sin CDN).
>                         > - [ ] - [ ] Cálculo opcional de HVL₁/HVL₂/Iₕ desde la propia tabla, sólo como verificación pedagógica.
>                         >
>                         > - [ ] ---
>                         >
>                         > - [ ] ## Cómo citar
>                         >
>                         > - [ ] > Tejada Castro, L. (2026). *Simulador de Atenuación de Rayos X (Beta 0.1)* [Software]. Departamento de Tecnología Médica, Universidad de Chile.
>                         >
>                         > - [ ] ---
>                         >
>                         > - [ ] ## Autor
>                         >
>                         > - [ ] **Luciano Tejada Castro**
>                         > - [ ] Departamento de Tecnología Médica · Facultad de Medicina · Universidad de Chile
>                         >
>                         > - [ ] ---
>                         >
>                         > - [ ] ## Licencia
>                         >
>                         > - [ ] Distribuido bajo licencia **MIT**. La calibración contra los datasets del práctico es válida específicamente para el contexto del **Laboratorio Práctico N°1** del curso *Protección Radiológica y Radiobiología I* y **no debe utilizarse como sustituto de mediciones reales** en aplicaciones clínicas, dosimétricas o regulatorias.
>                         > - [ ] 
