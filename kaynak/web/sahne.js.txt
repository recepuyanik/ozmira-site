// OZMIRA 3B sahnesi. Veriden bağımsız: createSahne(host) bir kontrolcü döndürür, veriyi setDurum(durum) ile alır.
// Durum biçimi: { merkez:{ad,renk}, sistemler:[{id,ad,stil,renk,ajanlar:[{ad,aciklama,calisma,token,kosuyor}]}] }
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';

export const STILLER = ['galaksi', 'lif', 'sarmal', 'karadelik', 'kure'];
export const fmt = (n) => (n >= 1e6 ? (n / 1e6).toFixed(1) + 'M' : n >= 1e3 ? (n / 1e3).toFixed(1) + 'k' : String(Math.round(n || 0)));
const rnd = (a = 0, b = 1) => a + Math.random() * (b - a);
const gauss = () => { let u = 0, v = 0; while (!u) u = Math.random(); while (!v) v = Math.random(); return Math.sqrt(-2 * Math.log(u)) * Math.cos(2 * Math.PI * v); };
const ease = (t) => (t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2);
const clamp = (x, a, b) => Math.max(a, Math.min(b, x));
const TAU = Math.PI * 2;
const SOFT_FRAG = 'varying vec3 vC; varying float vA; void main(){ float r = length(gl_PointCoord - 0.5); if (r > 0.5) discard; float a = 1.0 - smoothstep(0.32, 0.5, r); a *= 0.55 + 0.45 * (1.0 - smoothstep(0.0, 0.32, r)); gl_FragColor = vec4(vC, a * vA); }';

// Sistemlerin sahnedeki yeri: 1-4 sistem sol, sağ, üst, alt; 5 ve fazlası elips üzerinde eşit aralıklı (8'den sonra elips sistem sayısıyla büyür).
function konumlar(n) {
  const big = n >= 5, RX = big ? Math.max(11.5, 1.35 * n) : 9.5, RY = big ? Math.max(10, 1.2 * n) : 8.6, P = Math.PI;
  const sets = { 1: [P], 2: [P, 0], 3: [P, 0, P / 2], 4: [P, 0, P / 2, -P / 2] };
  const angs = sets[n] || Array.from({ length: n }, (_, i) => P - (i * TAU) / n);
  return angs.map((a) => ({ a, c: new THREE.Vector3(Math.cos(a) * RX, Math.sin(a) * RY, 0) }));
}

export function createSahne(host, opts = {}) {
  const reduced = matchMedia('(prefers-reduced-motion: reduce)').matches;
  const W0 = host.clientWidth || 800, H0 = host.clientHeight || 500;
  const V3 = THREE.Vector3;

  // ---------- Renderer, sahne, kamera, ışıma ----------
  const renderer = new THREE.WebGLRenderer({ antialias: false, powerPreference: 'high-performance' });
  let pixelRatio = Math.min(devicePixelRatio || 1, 2); const PR_MIN = Math.min(devicePixelRatio || 1, 1.5);
  renderer.setPixelRatio(pixelRatio); renderer.setSize(W0, H0);
  renderer.domElement.style.cssText = 'display:block;width:100%;height:100%;touch-action:none;cursor:grab';
  host.appendChild(renderer.domElement);
  const scene = new THREE.Scene(); scene.background = new THREE.Color(0x02030a);
  const camera = new THREE.PerspectiveCamera(40, W0 / H0, 0.1, 500); camera.position.set(0, 16, 76);
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true; controls.dampingFactor = 0.06; controls.minDistance = 3; controls.maxDistance = 160;
  controls.rotateSpeed = 0.55; controls.zoomSpeed = 0.7; controls.panSpeed = 0.5; controls.minPolarAngle = Math.PI * 0.08; controls.maxPolarAngle = Math.PI * 0.92;
  const rt = new THREE.WebGLRenderTarget(W0 * pixelRatio, H0 * pixelRatio, { type: THREE.HalfFloatType, samples: 4 });
  const composer = new EffectComposer(renderer, rt); composer.setPixelRatio(pixelRatio); composer.setSize(W0, H0);
  composer.addPass(new RenderPass(scene, camera));
  const bloom = new UnrealBloomPass(new THREE.Vector2(W0, H0), 0.12, 0.18, 0.82); composer.addPass(bloom); composer.addPass(new OutputPass());
  const BLOOM = 0.14; // 0.85, 0.48 ve 0.30 gerçek ekranda bulanık/hare gibi göründü; netlik için dar ve zayıf tutuldu

  // ---------- Ortak yardımcılar ----------
  const pxRefs = [];
  const pxU = () => { const u = { value: pixelRatio }; pxRefs.push(u); return u; };
  const glowTex = (() => { const c = document.createElement('canvas'); c.width = c.height = 128; const g = c.getContext('2d'), gr = g.createRadialGradient(64, 64, 0, 64, 64, 64);
    gr.addColorStop(0, 'rgba(255,255,255,1)'); gr.addColorStop(0.25, 'rgba(255,255,255,.55)'); gr.addColorStop(1, 'rgba(255,255,255,0)'); g.fillStyle = gr; g.fillRect(0, 0, 128, 128);
    const t = new THREE.CanvasTexture(c); t.colorSpace = THREE.SRGBColorSpace; return t; })();
  const spriteMat = (color, opacity = 1) => new THREE.SpriteMaterial({ map: glowTex, color, blending: THREE.AdditiveBlending, depthWrite: false, transparent: true, opacity });
  const add = (o, objs) => { scene.add(o); objs.push(o); return o; };
  const glowAt = (color, op, sx, sy, pos, objs) => { const s = new THREE.Sprite(spriteMat(color, op * 0.35)); s.scale.set(sx * 0.75, sy * 0.75, 1); s.position.copy(pos); return add(s, objs); };
  const cv = (col, k) => new V3(col.r * k, col.g * k, col.b * k);
  const WHITE = new THREE.Color(1, 1, 1);
  const zeros = (n) => new THREE.BufferAttribute(new Float32Array(n * 3), 3);
  const attr = (arr, n) => new THREE.BufferAttribute(arr, n);
  const pointsMat = (uniforms, vs) => new THREE.ShaderMaterial({ uniforms, vertexShader: vs, fragmentShader: SOFT_FRAG, transparent: true, depthWrite: false, blending: THREE.AdditiveBlending });
  const mkPoints = (geo, mat, parent) => { const p = new THREE.Points(geo, mat); p.frustumCulled = false; parent.add(p); return p; };
  const disposeObj = (o) => { o.traverse((x) => { if (x.geometry) x.geometry.dispose(); if (x.material) { (Array.isArray(x.material) ? x.material : [x.material]).forEach((m) => m.dispose()); } }); };

  // ---------- Yıldız alanı ----------
  const stars = (() => { const n = 2600, p = new Float32Array(n * 3), c = new Float32Array(n * 3);
    for (let i = 0; i < n; i++) { const r = rnd(90, 190), th = rnd(0, TAU), ph = Math.acos(rnd(-1, 1)); p[i * 3] = r * Math.sin(ph) * Math.cos(th); p[i * 3 + 1] = r * Math.cos(ph); p[i * 3 + 2] = r * Math.sin(ph) * Math.sin(th);
      const k = rnd(0.35, 0.9), warm = Math.random() < 0.15; c[i * 3] = warm ? k : k * 0.8; c[i * 3 + 1] = k * 0.85; c[i * 3 + 2] = warm ? k * 0.7 : k; }
    const g = new THREE.BufferGeometry(); g.setAttribute('position', attr(p, 3)); g.setAttribute('color', attr(c, 3));
    const s = new THREE.Points(g, new THREE.PointsMaterial({ size: 0.55, vertexColors: true, transparent: true, opacity: 0.8, depthWrite: false })); s.frustumCulled = false; scene.add(s); return s; })();

  // ---------- Merkez çekirdeği ----------
  const merkez = new THREE.Group(); scene.add(merkez);
  // OZMIRA işareti (Tünel): beyaz çekirdek, içinde dönen küçük galaksiler, etrafında logodaki gibi çapraz kayan halkalar.
  const mm = { core: new THREE.MeshBasicMaterial({ color: 0xffffff }), halo: spriteMat(0xffffff, 0.14), halo2: spriteMat(0xf3b84b, 0.06), rings: [] };
  merkez.add(new THREE.Mesh(new THREE.SphereGeometry(0.34, 32, 24), mm.core));
  { const h = new THREE.Sprite(mm.halo); h.scale.set(3.2, 3.2, 1); const h2 = new THREE.Sprite(mm.halo2); h2.scale.set(6, 6, 1); merkez.add(h, h2); }
  const halkaG = new THREE.Group(); merkez.add(halkaG);
  const HALKA = [[2.05, 0.3], [1.63, 0.5], [1.21, 0.7], [0.79, 0.88]].map(([r, op], i) => {
    const m = new THREE.MeshBasicMaterial({ color: 0xf3b84b, transparent: true, opacity: op, side: THREE.DoubleSide, blending: THREE.AdditiveBlending, depthWrite: false });
    const mesh = new THREE.Mesh(new THREE.RingGeometry(r - 0.045, r + 0.045, 128), m); halkaG.add(mesh); mm.rings.push(m); return { mesh, k: 4 - i }; });
  const icGal = [[1.0, 0.5, 0.2, 0.9], [1.45, -0.7, 1.1, -0.6], [0.7, 1.3, -0.4, 1.4]].map(([R, rx, rz, hiz]) => {
    const N = 520, pos = new Float32Array(N * 3), col = new Float32Array(N * 3);
    for (let i = 0; i < N; i++) { const r = Math.pow(Math.random(), 0.6) * R, a = (i % 2) * Math.PI + (r / R) * 6.4 + gauss() * 0.22, k = 0.75 + Math.random() * 0.25;
      pos.set([Math.cos(a) * r, gauss() * 0.03, Math.sin(a) * r], i * 3); col.set([k, k, k * (0.9 + 0.1 * Math.random())], i * 3); }
    const g = new THREE.BufferGeometry(); g.setAttribute('position', attr(pos, 3)); g.setAttribute('color', attr(col, 3));
    const pts = new THREE.Points(g, new THREE.PointsMaterial({ size: 0.14, vertexColors: true, transparent: true, opacity: 0.85, depthWrite: false })); pts.frustumCulled = false; pts.rotation.set(rx, 0, rz); merkez.add(pts); return { pts, hiz }; });
  function setMerkezRenk(hex) { const c = new THREE.Color(hex); mm.rings.forEach((m) => m.color.copy(c)); mm.halo2.color.copy(c); }
  function merkezGuncelle(t) {
    halkaG.quaternion.copy(camera.quaternion);
    HALKA.forEach((h) => { const d = (0.184 + 0.05 * Math.sin(t * 0.8 + h.k * 0.5)) * h.k; h.mesh.position.set(-d, -d, 0); });
    icGal.forEach((g) => { g.pts.rotation.y = t * 0.25 * g.hiz; });
  }

  // ---------- Görsel stiller ----------
  // Her stil { update(t), place(i,n) -> (t,out)=>out, dyn, titleY } döndürür. Nesneleri objs listesine ekler (silinirken temizlenir).
  const STIL = {
    galaksi(c, ang, col, objs) {
      const g = add(new THREE.Group(), objs); g.position.copy(c); g.rotation.set(0.45, 0, 0.15);
      const U = { uTime: { value: 0 }, uPx: pxU(), uSpeed: { value: reduced ? 0.03 : 0.12 } };
      const N = 14000, R = 5.5, aR = new Float32Array(N), aA = new Float32Array(N), aS = new Float32Array(N), aC = new Float32Array(N * 3), pos = new Float32Array(N * 3);
      const core = new THREE.Color(1.0, 0.82, 0.55), arm = col.clone().lerp(WHITE, 0.35).multiplyScalar(1.3), tmp = new THREE.Color();
      for (let i = 0; i < N; i++) { const r = Math.pow(Math.random(), 0.6) * R, t = r / R;
        aA[i] = (i % 2) * Math.PI + r * 1.25 + gauss() * (0.1 + (1 - t) * 0.35 + t * 0.12); aR[i] = r + gauss() * 0.06; pos[i * 3 + 1] = gauss() * 0.16 * (1 - t * 0.7);
        aS[i] = Math.random() < 0.03 ? rnd(2.2, 3.6) : rnd(0.6, 1.5); tmp.copy(core).lerp(arm, Math.min(1, t * 1.4)).multiplyScalar(0.55 + (1 - t) * 0.6); aC.set([tmp.r, tmp.g, tmp.b], i * 3); }
      const geo = new THREE.BufferGeometry(); geo.setAttribute('position', attr(pos, 3)); geo.setAttribute('aR', attr(aR, 1)); geo.setAttribute('aA', attr(aA, 1)); geo.setAttribute('aS', attr(aS, 1)); geo.setAttribute('aC', attr(aC, 3));
      mkPoints(geo, pointsMat(U, 'attribute float aR; attribute float aA; attribute float aS; attribute vec3 aC; uniform float uTime; uniform float uPx; uniform float uSpeed; varying vec3 vC; varying float vA; void main(){ float w = 1.6 / (sqrt(aR) + 0.6); float a = aA + uTime * uSpeed * w; vec3 p = vec3(cos(a) * aR, position.y, sin(a) * aR); vec4 mv = modelViewMatrix * vec4(p, 1.0); gl_PointSize = aS * uPx * (28.0 / -mv.z); gl_Position = projectionMatrix * mv; vC = aC; vA = 1.0; }'), g);
      const gl = new THREE.Sprite(spriteMat(col, 0.26)); gl.scale.set(5, 5, 1); g.add(gl);
      return { titleY: -7.4, dyn: false, update(t) { U.uTime.value = t; },
        place(i, n) { const a = (i / n) * TAU + 0.3, r = 6.6 + (i % 2) * 0.9; g.updateMatrixWorld(true); const w = g.localToWorld(new V3(Math.cos(a) * r, 0, Math.sin(a) * r)); return (t, out) => out.copy(w); } };
    },

    lif(c, ang, col, objs) {
      const U = { uTime: { value: 0 }, uC1: { value: cv(col, 1.5) }, uC2: { value: cv(col.clone().lerp(WHITE, 0.55), 1.4) } };
      const F = 320, S = 26, RX = 2.6, RY = 4.0, RZ = 2.0, pos = new Float32Array(F * S * 6), aT = new Float32Array(F * S * 2), aP = new Float32Array(F * S * 2);
      const surf = () => { const v = new V3(gauss(), gauss() * 1.2, gauss()).normalize(), k = rnd(0.7, 1); return v.set(v.x * RX * k, v.y * RY * k, v.z * RZ * k); };
      const A = new V3(), B = new V3(), Cc = new V3(), P0 = new V3(), P1 = new V3();
      const bez = (t, o) => { const u = 1 - t; return o.set(u * u * A.x + 2 * u * t * Cc.x + t * t * B.x, u * u * A.y + 2 * u * t * Cc.y + t * t * B.y, u * u * A.z + 2 * u * t * Cc.z + t * t * B.z); };
      let o = 0;
      for (let f = 0; f < F; f++) { A.copy(surf()); B.copy(surf()); Cc.copy(A).add(B).multiplyScalar(0.18).add(new V3(gauss() * 0.5, gauss() * 0.9, gauss() * 0.5)); const ph = Math.random();
        for (let s = 0; s < S; s++) { bez(s / S, P0); bez((s + 1) / S, P1); pos.set([P0.x, P0.y, P0.z, P1.x, P1.y, P1.z], o * 3); aT[o] = s / S; aT[o + 1] = (s + 1) / S; aP[o] = aP[o + 1] = ph; o += 2; } }
      const geo = new THREE.BufferGeometry(); geo.setAttribute('position', attr(pos, 3)); geo.setAttribute('aT', attr(aT, 1)); geo.setAttribute('aP', attr(aP, 1));
      const lines = new THREE.LineSegments(geo, new THREE.ShaderMaterial({ uniforms: U, transparent: true, depthWrite: false, blending: THREE.AdditiveBlending,
        vertexShader: 'attribute float aT; attribute float aP; varying float vT; varying float vP; varying float vY; void main(){ vT = aT; vP = aP; vY = position.y; gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0); }',
        fragmentShader: 'uniform float uTime; uniform vec3 uC1; uniform vec3 uC2; varying float vT; varying float vP; varying float vY; void main(){ float flow = pow(0.5 + 0.5 * sin(vT * 9.42 - uTime * 1.6 + vP * 6.283), 4.0); float scanY = mod(uTime * 1.3, 10.0) - 5.0; float scan = exp(-pow((vY - scanY) * 1.3, 2.0)); float a = 0.10 + 0.75 * flow + 0.9 * scan; vec3 col = mix(uC1, uC2, clamp(flow * 0.8 + scan, 0.0, 1.0)); gl_FragColor = vec4(col, a * 0.5); }' }));
      lines.frustumCulled = false; lines.position.copy(c); add(lines, objs); glowAt(col, 0.12, 9, 11, c, objs);
      return { titleY: -6.8, dyn: false, update(t) { U.uTime.value = t; },
        place(i, n) { const y = 1 - (i / Math.max(1, n - 1)) * 2, rad = Math.sqrt(1 - y * y), th = i * 2.399963, w = new V3(Math.cos(th) * rad * 3.7, y * 5.4, Math.sin(th) * rad * 2.9).add(c); return (t, out) => out.copy(w); } };
    },

    sarmal(c, ang, col, objs) {
      const GEM = { L: 9.5, R: 1.5, turns: 2.6 }, hsl = { h: 0, s: 0, l: 0 };
      col.getHSL(hsl); const col2 = new THREE.Color().setHSL((hsl.h + 0.14) % 1, clamp(hsl.s, 0.3, 1), clamp(hsl.l, 0.5, 0.7));
      const U = { uPh: { value: 0 }, uL: { value: GEM.L }, uR: { value: GEM.R }, uTurns: { value: GEM.turns }, uPx: pxU(), uGlow: { value: 1 },
        uC1: { value: cv(col.clone().lerp(WHITE, 0.2), 1.5) }, uC2: { value: cv(col2, 1.5) }, uC3: { value: cv(col.clone().lerp(col2, 0.5).lerp(WHITE, 0.35), 1.2) } };
      const g = add(new THREE.Group(), objs); g.position.copy(c); g.rotation.set(0.5, 0, ang - Math.PI / 2 - 0.22);
      const HELIX = 'uniform float uPh; uniform float uL; uniform float uR; uniform float uTurns; vec3 helix(float u, float s){ float env = 0.3 + 0.7 * sin(3.14159265 * clamp(u, 0.001, 1.0)); float a = u * uTurns * 6.2831853 + s * 3.14159265 + uPh; return vec3((u - 0.5) * uL, uR * env * cos(a), uR * env * sin(a)); }';
      const helixJS = (u, s, ph, out) => { const env = 0.3 + 0.7 * Math.sin(Math.PI * clamp(u, 0.001, 1)), a = u * GEM.turns * TAU + s * Math.PI + ph; return out.set((u - 0.5) * GEM.L, GEM.R * env * Math.cos(a), GEM.R * env * Math.sin(a)); };
      { const N = 700, n = N * 2, aU = new Float32Array(n), aS = new Float32Array(n), aZ = new Float32Array(n);
        for (let i = 0; i < n; i++) { aU[i] = ((i % N) + Math.random() * 0.6) / N; aS[i] = i < N ? 0 : 1; aZ[i] = rnd(1.0, 2.4); }
        const geo = new THREE.BufferGeometry(); geo.setAttribute('position', zeros(n)); geo.setAttribute('aU', attr(aU, 1)); geo.setAttribute('aS', attr(aS, 1)); geo.setAttribute('aZ', attr(aZ, 1));
        mkPoints(geo, pointsMat(U, HELIX + ' uniform vec3 uC1; uniform vec3 uC2; attribute float aU; attribute float aS; attribute float aZ; uniform float uPx; uniform float uGlow; varying vec3 vC; varying float vA; void main(){ vec3 p = helix(aU, aS); vec4 mv = modelViewMatrix * vec4(p, 1.0); gl_PointSize = aZ * uPx * (30.0 / -mv.z); gl_Position = projectionMatrix * mv; vC = aS < 0.5 ? uC1 : uC2; vA = (0.55 + 0.45 * sin(aU * 20.0 - uPh * 3.0)) * uGlow; }'), g); }
      { const D = 300, dU = new Float32Array(D), dA = new Float32Array(D), dD = new Float32Array(D), dZ = new Float32Array(D);
        for (let i = 0; i < D; i++) { dU[i] = Math.random(); dA[i] = rnd(0, TAU); dD[i] = gauss() * 0.9; dZ[i] = rnd(0.7, 1.6); }
        const geo = new THREE.BufferGeometry(); geo.setAttribute('position', zeros(D)); geo.setAttribute('aU', attr(dU, 1)); geo.setAttribute('aA', attr(dA, 1)); geo.setAttribute('aD', attr(dD, 1)); geo.setAttribute('aZ', attr(dZ, 1));
        mkPoints(geo, pointsMat(U, 'uniform float uPh; uniform float uL; uniform float uR; uniform float uPx; uniform float uGlow; uniform vec3 uC1; uniform vec3 uC2; attribute float aU; attribute float aA; attribute float aD; attribute float aZ; varying vec3 vC; varying float vA; void main(){ float env = 0.3 + 0.7 * sin(3.14159265 * clamp(aU, 0.001, 1.0)); float a = aA + uPh * 0.35; vec3 p = vec3((aU - 0.5) * uL + sin(aA * 3.0) * 0.2, (uR * env + aD) * cos(a), (uR * env + aD) * sin(a)); vec4 mv = modelViewMatrix * vec4(p, 1.0); gl_PointSize = aZ * uPx * (24.0 / -mv.z); gl_Position = projectionMatrix * mv; vC = mix(uC1, uC2, fract(aA)); vA = 0.55 * uGlow; }'), g); }
      { const K = 36, ru = new Float32Array(K * 2), rs = new Float32Array(K * 2);
        for (let k = 0; k < K; k++) { ru[k * 2] = ru[k * 2 + 1] = (k + 0.5) / K; rs[k * 2] = 0; rs[k * 2 + 1] = 1; }
        const geo = new THREE.BufferGeometry(); geo.setAttribute('position', zeros(K * 2)); geo.setAttribute('aU', attr(ru, 1)); geo.setAttribute('aS', attr(rs, 1));
        const rungs = new THREE.LineSegments(geo, new THREE.ShaderMaterial({ uniforms: U, transparent: true, depthWrite: false, blending: THREE.AdditiveBlending,
          vertexShader: HELIX + ' attribute float aU; attribute float aS; uniform float uGlow; varying float vA; void main(){ gl_Position = projectionMatrix * modelViewMatrix * vec4(helix(aU, aS), 1.0); vA = 0.2 * uGlow; }',
          fragmentShader: 'uniform vec3 uC3; varying float vA; void main(){ gl_FragColor = vec4(uC3, vA); }' })); rungs.frustumCulled = false; g.add(rungs); }
      glowAt(col, 0.16, 9, 5.5, c, objs);
      const tmp = new V3();
      return { titleY: -3.4, dyn: true, update(t) { U.uPh.value = t * (reduced ? 0.1 : 0.55); },
        place(i, n) { const u = (i + 1) / (n + 1), s = i % 2; return (t, out) => { helixJS(u, s, U.uPh.value, tmp); g.updateMatrixWorld(); return out.copy(g.localToWorld(tmp)); }; } };
    },

    karadelik(c, ang, col, objs) {
      const g = add(new THREE.Group(), objs); g.position.copy(c); g.rotation.set(0.42, 0, 0.12);
      const U = { uTime: { value: 0 }, uPx: pxU(), uAcc: { value: cv(col, 1.5) } };
      const horizon = new THREE.Mesh(new THREE.SphereGeometry(0.92, 48, 32), new THREE.MeshBasicMaterial({ color: 0x000000 })); horizon.position.copy(c); add(horizon, objs);
      const ringA = new THREE.Mesh(new THREE.RingGeometry(0.97, 1.05, 160), new THREE.MeshBasicMaterial({ color: col.clone().multiplyScalar(1.3), transparent: true, opacity: 0.8, blending: THREE.AdditiveBlending, depthWrite: false, side: THREE.DoubleSide }));
      const ringB = new THREE.Mesh(new THREE.RingGeometry(1.3, 1.62, 160), new THREE.MeshBasicMaterial({ color: 0xe5e7eb, transparent: true, opacity: 0.14, blending: THREE.AdditiveBlending, depthWrite: false, side: THREE.DoubleSide }));
      ringA.position.copy(c); ringB.position.copy(c); add(ringA, objs); add(ringB, objs);
      { const N = 6500, aR = new Float32Array(N), aA = new Float32Array(N), aZ = new Float32Array(N), pos = new Float32Array(N * 3);
        for (let i = 0; i < N; i++) { aR[i] = 1.35 * Math.pow(4.6 / 1.35, Math.random()); aA[i] = rnd(0, TAU); aZ[i] = rnd(0.6, 1.7); pos[i * 3 + 1] = gauss() * 0.05 * aR[i]; }
        const geo = new THREE.BufferGeometry(); geo.setAttribute('position', attr(pos, 3)); geo.setAttribute('aR', attr(aR, 1)); geo.setAttribute('aA', attr(aA, 1)); geo.setAttribute('aZ', attr(aZ, 1));
        mkPoints(geo, pointsMat(U, 'attribute float aR; attribute float aA; attribute float aZ; uniform float uTime; uniform float uPx; uniform vec3 uAcc; varying vec3 vC; varying float vA; void main(){ float a = aA + uTime * 1.5 / pow(aR, 1.5); vec3 p = vec3(cos(a) * aR, position.y, sin(a) * aR); vec4 mv = modelViewMatrix * vec4(p, 1.0); gl_PointSize = aZ * uPx * (26.0 / -mv.z); gl_Position = projectionMatrix * mv; float heat = 1.0 - smoothstep(1.4, 4.6, aR); vec3 hot = vec3(0.88, 1.0, 0.98); vec3 cold = vec3(0.72, 0.75, 0.8); vC = mix(cold, mix(uAcc, hot, heat), heat) * (0.55 + 0.9 * heat); vA = 0.65 + 0.55 * sin(a + 0.6); }'), g);
        const J = 900, jT = new Float32Array(J), jS = new Float32Array(J), jR = new Float32Array(J), jA = new Float32Array(J);
        for (let i = 0; i < J; i++) { jT[i] = Math.random(); jS[i] = i % 2 ? 1 : -1; jR[i] = rnd(0.3, 1); jA[i] = rnd(0, TAU); }
        const geo2 = new THREE.BufferGeometry(); geo2.setAttribute('position', zeros(J)); geo2.setAttribute('aT', attr(jT, 1)); geo2.setAttribute('aSd', attr(jS, 1)); geo2.setAttribute('aRn', attr(jR, 1)); geo2.setAttribute('aA', attr(jA, 1));
        mkPoints(geo2, pointsMat(U, 'attribute float aT; attribute float aSd; attribute float aRn; attribute float aA; uniform float uTime; uniform float uPx; uniform vec3 uAcc; varying vec3 vC; varying float vA; void main(){ float t = fract(aT + uTime * 0.28); float rad = 0.05 + t * 0.5 * aRn; vec3 p = vec3(cos(aA) * rad, aSd * (0.95 + t * 6.2), sin(aA) * rad); vec4 mv = modelViewMatrix * vec4(p, 1.0); gl_PointSize = uPx * (20.0 / -mv.z); gl_Position = projectionMatrix * mv; vC = mix(uAcc, vec3(0.9, 1.0, 1.0), 1.0 - t); vA = (1.0 - t) * smoothstep(0.0, 0.08, t) * 0.8; }'), g); }
      glowAt(0xe5e7eb, 0.1, 9, 9, c, objs);
      const tmp = new V3();
      return { titleY: 5.4, dyn: true, update(t) { U.uTime.value = t; ringA.quaternion.copy(camera.quaternion); ringB.quaternion.copy(camera.quaternion); },
        place(i, n) { const r = n <= 1 ? 3.3 : 2.4 + (i * 2.0) / (n - 1), th0 = rnd(0, TAU); return (t, out) => { const a = th0 + (t * 1.5) / Math.pow(r, 1.5); tmp.set(Math.cos(a) * r, 0, Math.sin(a) * r); g.updateMatrixWorld(); return out.copy(g.localToWorld(tmp)); }; } };
    },

    kure(c, ang, col, objs) {
      const g = add(new THREE.Group(), objs); g.position.copy(c);
      const U = { uTime: { value: 0 }, uPx: pxU(), uC1: { value: cv(col.clone().lerp(WHITE, 0.3), 1.5) } };
      g.add(new THREE.Mesh(new THREE.SphereGeometry(0.5, 32, 24), new THREE.MeshBasicMaterial({ color: col.clone().multiplyScalar(1.3) })));
      g.add(new THREE.Mesh(new THREE.IcosahedronGeometry(0.85, 2), new THREE.MeshBasicMaterial({ color: col, wireframe: true, transparent: true, opacity: 0.3, blending: THREE.AdditiveBlending, depthWrite: false })));
      const rm = new THREE.MeshBasicMaterial({ color: col, transparent: true, opacity: 0.7, side: THREE.DoubleSide, blending: THREE.AdditiveBlending, depthWrite: false });
      const r1 = new THREE.Mesh(new THREE.RingGeometry(1.35, 1.39, 96), rm), r2 = new THREE.Mesh(new THREE.RingGeometry(1.8, 1.83, 96), rm.clone());
      r1.rotation.set(1.2, 0.2, 0); r2.rotation.set(0.5, 0.6, 0); r2.material.opacity = 0.4; g.add(r1, r2);
      { const N = 1100, pos = new Float32Array(N * 3), aK = new Float32Array(N), aZ = new Float32Array(N);
        for (let i = 0; i < N; i++) { const v = new V3(gauss(), gauss(), gauss()).normalize().multiplyScalar(2.3 + gauss() * 0.15); pos.set([v.x, v.y, v.z], i * 3); aK[i] = Math.random(); aZ[i] = rnd(0.7, 1.6); }
        const geo = new THREE.BufferGeometry(); geo.setAttribute('position', attr(pos, 3)); geo.setAttribute('aK', attr(aK, 1)); geo.setAttribute('aZ', attr(aZ, 1));
        mkPoints(geo, pointsMat(U, 'attribute float aK; attribute float aZ; uniform float uTime; uniform float uPx; uniform vec3 uC1; varying vec3 vC; varying float vA; void main(){ float a = uTime * 0.12 * (0.5 + aK); float s = sin(a), co = cos(a); vec3 p = vec3(co * position.x - s * position.z, position.y, s * position.x + co * position.z); vec4 mv = modelViewMatrix * vec4(p, 1.0); gl_PointSize = aZ * uPx * (24.0 / -mv.z); gl_Position = projectionMatrix * mv; vC = uC1; vA = 0.5 + 0.4 * aK; }'), g); }
      glowAt(col, 0.2, 6, 6, c, objs);
      return { titleY: -3.6, dyn: true, update(t) { U.uTime.value = t; },
        place(i, n) { const r = 3.5 + (i % 3) * 0.5, th0 = (i / n) * TAU, inc = rnd(-0.9, 0.9), sp = 0.3 * (1 + (i % 3) * 0.2); return (t, out) => { const a = th0 + t * sp; return out.set(c.x + Math.cos(a) * r, c.y + Math.sin(a) * r * Math.sin(inc), c.z + Math.sin(a) * r * Math.cos(inc)); }; } };
    },
  };

  // ---------- Etiketler ----------
  const overlay = document.createElement('div'); overlay.style.cssText = 'position:absolute;inset:0;pointer-events:none;overflow:hidden;font:500 11px/1.2 "SF Pro Display","Inter","Segoe UI",system-ui,sans-serif;letter-spacing:.08em;text-transform:uppercase';
  host.appendChild(overlay);
  const labels = new Map();
  function mkLabel(key, text, pos, kind, off) { const el = document.createElement('div'); el.textContent = text;
    el.style.cssText = 'position:absolute;left:0;top:0;white-space:nowrap;opacity:0;transition:opacity .25s;padding:2px 6px;border-radius:3px;will-change:transform;' + (kind === 'title' ? 'font-size:12px;letter-spacing:.22em;color:#e8ecf4;' : 'color:#aab3c5;background:rgba(6,8,14,.55);backdrop-filter:blur(4px);');
    overlay.appendChild(el); const L = { el, pos, off: off || new V3(), kind, w: 0, h: 0, dirty: true, shown: false, want: false }; labels.set(key, L); return L; }
  const merkezEtiket = mkLabel('t:merkez', 'OZMIRA', new V3(0, -2.9, 0), 'title');

  // ---------- Durum: sistemler ve ajanlar ----------
  const systems = new Map(), agents = new Map(), presets = { hepsi: { pos: new V3(0, 1.5, 37), tgt: new V3() }, merkez: { pos: new V3(0, 1, 7), tgt: new V3() } };
  let sig = '', now = 0, selected = null, onSelectCb = opts.onSelect || null, firstBuild = true;
  const tmpV = new V3(), linePts = new V3();

  function clearAll() {
    agents.forEach((a) => { scene.remove(a.sprite); a.sprite.material.dispose(); scene.remove(a.line); a.line.geometry.dispose(); a.line.material.dispose(); });
    agents.clear(); signals.forEach((s) => s.sp.forEach((x) => scene.remove(x))); signals.length = 0;
    systems.forEach((s) => s.objs.forEach((o) => { scene.remove(o); disposeObj(o); })); systems.clear(); pxRefs.length = 0; selected = null;
    [...labels.keys()].forEach((k) => { if (k !== 't:merkez') { labels.get(k).el.remove(); labels.delete(k); } });
    Object.keys(presets).forEach((k) => { if (k !== 'hepsi' && k !== 'merkez') delete presets[k]; });
  }
  function fixCurve(a) { a.curve.v2.copy(a.pos); a.curve.v1.copy(a.pos).multiplyScalar(0.5).add(tmpV.set(0, 1.4, 0.9)); }
  function refreshLine(a) { const p = a.line.geometry.attributes.position; for (let i = 0; i <= 24; i++) { a.curve.getPoint(i / 24, linePts); p.setXYZ(i, linePts.x, linePts.y, linePts.z); } p.needsUpdate = true; }
  function addAgent(sys, rec, i, n) {
    const a = { id: sys.id + ':' + rec.ad, sys, name: rec.ad, desc: rec.aciklama || '', runs: rec.calisma || 0, tokens: rec.token || 0, running: !!rec.kosuyor, nextPulse: 0, pulses: 0, flash: 0, flashText: '', pos: new V3(), sx: 0, sy: 0, vis: false, dyn: sys.b.dyn };
    a.fn = sys.b.place(i, n); a.fn(0, a.pos);
    a.sprite = new THREE.Sprite(spriteMat(sys.col, 0.9)); a.sprite.scale.set(0.34, 0.34, 1); a.sprite.position.copy(a.pos); scene.add(a.sprite);
    a.curve = new THREE.QuadraticBezierCurve3(new V3(), new V3(), a.pos.clone()); fixCurve(a);
    a.line = new THREE.Line(new THREE.BufferGeometry().setFromPoints(a.curve.getPoints(24)), new THREE.LineBasicMaterial({ color: sys.col, transparent: true, opacity: 0.07, blending: THREE.AdditiveBlending, depthWrite: false })); scene.add(a.line);
    mkLabel('a:' + a.id, a.name, a.pos, 'agent', new V3(0, 0.5, 0)); agents.set(a.id, a);
  }
  function rebuild(d) {
    clearAll(); setMerkezRenk(d.merkez.renk); merkezEtiket.el.textContent = d.merkez.ad.toUpperCase(); merkezEtiket.dirty = true;
    const n = d.sistemler.length, pos = konumlar(n); presets.hepsi.pos.set(0, 1.5, n >= 5 ? Math.max(46, (Math.max(10, 1.2 * n) + 5) / 0.364) : 37);
    d.sistemler.forEach((s, i) => {
      const { a, c } = pos[i], col = new THREE.Color(s.renk), objs = [], b = STIL[s.stil](c, a, col, objs);
      const sys = { id: s.id, ad: s.ad, stil: s.stil, col, c, objs, b }; systems.set(s.id, sys);
      const dl = new THREE.Line(new THREE.BufferGeometry().setFromPoints([new V3(), c]), new THREE.LineDashedMaterial({ color: col, dashSize: 0.3, gapSize: 0.25, transparent: true, opacity: 0.16 })); dl.computeLineDistances(); add(dl, objs);
      presets[s.id] = { pos: c.clone().add(new V3(0, 3.4, 12.5)), tgt: c.clone() };
      mkLabel('t:' + s.id, s.ad.toUpperCase(), c.clone().add(new V3(0, b.titleY, 0)), 'title');
      s.ajanlar.forEach((rec, k) => addAgent(sys, rec, k, s.ajanlar.length));
    });
    if (firstBuild && !reduced) { flyTo(presets.hepsi.pos, presets.hepsi.tgt, 3400); } else { bloom.strength = BLOOM; flyTo(presets.hepsi.pos, presets.hepsi.tgt, 1200); }
    firstBuild = false;
  }
  function setDurum(d) {
    if (!d || d.yok || !d.merkez || !Array.isArray(d.sistemler) || !d.sistemler.length) return;
    const s2 = JSON.stringify([d.merkez, d.sistemler.map((s) => [s.id, s.ad, s.stil, s.renk, s.ajanlar.map((a) => a.ad)])]);
    if (s2 !== sig) { sig = s2; rebuild(d); }
    d.sistemler.forEach((s) => s.ajanlar.forEach((r) => { const a = agents.get(s.id + ':' + r.ad); if (!a) return;
      const dTok = (r.token || 0) - a.tokens, dRun = (r.calisma || 0) - a.runs;
      if (dTok > 0 || dRun > 0) pulse(a, { tokens: Math.max(0, dTok), ms: 500 });
      a.tokens = r.token || 0; a.runs = r.calisma || 0; a.desc = r.aciklama || a.desc; a.running = !!r.kosuyor; }));
  }

  // ---------- Sinyaller: merkez -> ajan -> (çalışır) -> merkez ----------
  const signals = [], sigMats = {}; let voiceAmp = 0;
  const sigMat = (sys, i) => (sigMats[sys.id + i + sys.col.getHex()] ||= spriteMat(sys.col, 1 - i * 0.28));
  function pulse(a, o = {}) {
    const sp = [0, 1, 2, 3].map((i) => { const s = new THREE.Sprite(sigMat(a.sys, i)); s.scale.setScalar(0.5 - i * 0.08); scene.add(s); return s; });
    a.pulses++; signals.push({ a, sp, phase: 'out', t: 0, until: 0, ms: o.ms ?? 2200, tokens: o.tokens || 0 });
  }
  function stepSignals(dt) {
    for (let i = signals.length - 1; i >= 0; i--) { const s = signals[i];
      if (s.phase === 'out') { s.t += dt / 1.1; if (s.t >= 1) { s.t = 1; s.phase = 'work'; s.until = now + s.ms / 1000; } }
      else if (s.phase === 'work') { if (now >= s.until) s.phase = 'back'; }
      else { s.t -= dt; if (s.t <= 0) { s.sp.forEach((x) => scene.remove(x)); s.a.pulses = Math.max(0, s.a.pulses - 1); voiceAmp = Math.min(1, voiceAmp + 0.3);
        if (s.tokens) { s.a.flash = now + 2.4; s.a.flashText = '+' + fmt(s.tokens); const L = labels.get('a:' + s.a.id); if (L) { L.el.textContent = s.a.name + '  ' + s.a.flashText; L.dirty = true; } }
        signals.splice(i, 1); continue; } }
      const dir = s.phase === 'back' ? 1 : -1; s.sp.forEach((x, k) => { s.a.curve.getPoint(clamp(ease(s.t) + dir * 0.035 * k, 0, 1), x.position); x.visible = s.phase !== 'work'; }); }
  }

  // ---------- Kamera ----------
  let tween = null;
  const flyTo = (pos, tgt, dur = 1400) => { tween = { t0: now, dur: dur / 1000, p0: camera.position.clone(), g0: controls.target.clone(), p1: pos.clone(), g1: tgt.clone() }; };
  function focus(name) { const p = presets[name]; if (p) flyTo(p.pos, p.tgt); }
  function focusAgent(a) { const dir = camera.position.clone().sub(a.pos).normalize(); flyTo(a.pos.clone().add(dir.multiplyScalar(6.5)), a.pos); }
  let sonDokunma = 0;
  controls.addEventListener('start', () => { tween = null; sonDokunma = now; controls.autoRotate = false; });
  controls.addEventListener('end', () => { sonDokunma = now; });
  if (reduced) { camera.position.copy(presets.hepsi.pos); bloom.strength = BLOOM; }

  // ---------- Fare: en yakın ajan ----------
  const ndc = new THREE.Vector2(9, 9), ptr = new THREE.Vector2(), ZERO2 = new THREE.Vector2(); let hovered = null, down = null; const mouse = { x: -999, y: -999 };
  const el = renderer.domElement;
  const nearest = (rad) => { let best = null, bd = rad * rad; agents.forEach((a) => { if (!a.vis) return; const d = (a.sx - mouse.x) ** 2 + (a.sy - mouse.y) ** 2; if (d < bd) { bd = d; best = a; } }); return best; };
  el.addEventListener('pointermove', (e) => { const r = el.getBoundingClientRect(); mouse.x = e.clientX - r.left; mouse.y = e.clientY - r.top; ndc.set((mouse.x / r.width) * 2 - 1, -(mouse.y / r.height) * 2 + 1); });
  el.addEventListener('pointerleave', () => { ndc.set(9, 9); mouse.x = mouse.y = -999; });
  el.addEventListener('pointerdown', (e) => { down = { x: e.clientX, y: e.clientY }; const r = el.getBoundingClientRect(); mouse.x = e.clientX - r.left; mouse.y = e.clientY - r.top; });
  el.addEventListener('pointerup', (e) => { if (down && Math.hypot(e.clientX - down.x, e.clientY - down.y) < 6) { const a = nearest(e.pointerType === 'touch' ? 28 : 16); if (a) { selected = a; focusAgent(a); if (onSelectCb) onSelectCb(a); } } down = null; });

  // ---------- Etiket yerleşimi ----------
  const tv = new V3(), tv2 = new V3();
  function updateLabels() {
    const W = host.clientWidth, H = host.clientHeight, cands = [], near = {}; systems.forEach((s, id) => { near[id] = camera.position.distanceTo(s.c) < 18; });
    labels.forEach((L, key) => { let prio = 0;
      if (L.kind === 'title') prio = 100;
      else { const a = agents.get(key.slice(2)); if (!a) { L.want = false; return; }
        if (a === hovered || a === selected) prio = 95; else if (a.running || a.pulses || a.flash > now) prio = 85; else if (near[a.sys.id]) prio = 40;
        if (a.flashText && a.flash <= now) { a.flashText = ''; L.el.textContent = a.name; L.dirty = true; } }
      L.prio = prio; if (prio) cands.push(L); else L.want = false; });
    cands.sort((a, b) => b.prio - a.prio); const placed = [];
    for (const L of cands) { tv.copy(L.pos).add(L.off).project(camera); if (tv.z > 1 || tv.z < -1) { L.want = false; continue; }
      if (L.dirty) { L.w = L.el.offsetWidth; L.h = L.el.offsetHeight; L.dirty = false; }
      const x = (tv.x * 0.5 + 0.5) * W, y = (-tv.y * 0.5 + 0.5) * H, r = { x0: x - L.w / 2 - 4, x1: x + L.w / 2 + 4, y0: y - L.h / 2 - 3, y1: y + L.h / 2 + 3 };
      if (placed.some((p) => r.x0 < p.x1 && r.x1 > p.x0 && r.y0 < p.y1 && r.y1 > p.y0) || x < 0 || x > W || y < 0 || y > H) { L.want = false; continue; }
      placed.push(r); L.want = true; L.el.style.transform = 'translate3d(' + x.toFixed(1) + 'px,' + y.toFixed(1) + 'px,0) translate(-50%,-50%)'; }
    labels.forEach((L) => { if (L.shown !== L.want) { L.el.style.opacity = L.want ? '1' : '0'; L.shown = L.want; } });
  }

  // ---------- Demo sinyalleri ----------
  let demoTimer = 0;
  function demo(on) { clearInterval(demoTimer); if (on) demoTimer = setInterval(() => { const list = [...agents.values()]; if (!list.length) return; pulse(list[(Math.random() * list.length) | 0], { tokens: Math.round(rnd(300, 4000)), ms: rnd(1400, 3200) }); }, 1300); }

  // ---------- Döngü ----------
  const clock = new THREE.Clock(); let raf = 0, visible = true, tabVisible = !document.hidden, frame = 0, avg = 0.016, destroyed = false;
  function applyPR(pr) { pixelRatio = pr; renderer.setPixelRatio(pr); composer.setPixelRatio(pr); composer.setSize(host.clientWidth, host.clientHeight); pxRefs.forEach((u) => { u.value = pr; }); }
  function tick() {
    raf = 0; if (destroyed || !visible || !tabVisible) return;
    const dt = Math.min(clock.getDelta(), 0.05); now += dt; frame++;
    if (tween) { const k = clamp((now - tween.t0) / tween.dur, 0, 1), e = ease(k); camera.position.lerpVectors(tween.p0, tween.p1, e); controls.target.lerpVectors(tween.g0, tween.g1, e); if (tween.dur > 3) bloom.strength = 0.12 + (BLOOM - 0.12) * e; if (k >= 1) { bloom.strength = BLOOM; tween = null; } }
    if (!reduced && !tween) { const bos = now - sonDokunma > 7; controls.autoRotate = bos; controls.autoRotateSpeed += ((bos ? 0.28 : 0) - controls.autoRotateSpeed) * 0.02; }
    controls.update();
    ptr.lerp(ndc.x > 5 ? ZERO2 : ndc, 0.04); stars.position.set(ptr.x * 1.2, ptr.y * 0.8, 0);
    voiceAmp *= Math.pow(0.04, dt); merkez.scale.setScalar(1 + 0.03 * Math.sin(now * 1.5) + voiceAmp * 0.16); merkezGuncelle(now);
    systems.forEach((s) => s.b.update(now));
    const W = host.clientWidth, H = host.clientHeight;
    agents.forEach((a) => {
      if (a.dyn) { a.fn(now, a.pos); a.sprite.position.copy(a.pos); fixCurve(a); if (frame % 2 === 0) refreshLine(a); }
      if (a.running && now >= a.nextPulse) { a.nextPulse = now + 3; pulse(a, { ms: 1400 }); }
      const hov = a === hovered, act = a.running || a.pulses > 0, tgt = act ? 0.62 + 0.1 * Math.sin(now * 8) : hov || a === selected ? 0.52 : 0.34, s = a.sprite.scale.x + (tgt - a.sprite.scale.x) * 0.15; a.sprite.scale.set(s, s, 1);
      a.line.material.opacity += ((act ? 0.5 : hov ? 0.3 : 0.07) - a.line.material.opacity) * 0.12;
      if (frame % 2 === 0) { tv2.copy(a.pos).project(camera); a.vis = tv2.z < 1; a.sx = (tv2.x * 0.5 + 0.5) * W; a.sy = (-tv2.y * 0.5 + 0.5) * H; } });
    stepSignals(dt);
    if (frame % 2 === 0) { hovered = nearest(16); el.style.cursor = hovered ? 'pointer' : 'grab'; }
    updateLabels(); composer.render();
    avg += (dt - avg) * 0.05;
    if (frame % 90 === 0 && frame > 120) { if (avg > 0.026 && pixelRatio > PR_MIN) applyPR(Math.max(PR_MIN, pixelRatio - 0.25)); else if (avg > 0.034 && bloom.enabled) bloom.enabled = false; }
    raf = requestAnimationFrame(tick);
  }
  const start = () => { if (!raf && !destroyed) { clock.getDelta(); raf = requestAnimationFrame(tick); } };
  const io = new IntersectionObserver((e) => { visible = e[0].isIntersecting; if (visible) start(); }); io.observe(host);
  const onVis = () => { tabVisible = !document.hidden; if (tabVisible) start(); }; document.addEventListener('visibilitychange', onVis);
  const ro = new ResizeObserver(() => { const w = host.clientWidth, h = host.clientHeight; if (!w || !h) return; renderer.setSize(w, h); composer.setSize(w, h); camera.aspect = w / h; camera.updateProjectionMatrix(); labels.forEach((L) => { L.dirty = true; }); }); ro.observe(host);
  setMerkezRenk('#f5b82e'); start();

  return {
    setDurum, focus, demo,
    onSelect(cb) { onSelectCb = cb; },
    secili() { return selected; },
    temizleSecim() { selected = null; },
    sistemler() { return [...systems.values()].map((s) => ({ id: s.id, ad: s.ad })); },
    destroy() { destroyed = true; cancelAnimationFrame(raf); clearInterval(demoTimer); io.disconnect(); ro.disconnect(); document.removeEventListener('visibilitychange', onVis); clearAll(); controls.dispose(); composer.dispose(); renderer.dispose(); renderer.domElement.remove(); overlay.remove(); },
  };
}
