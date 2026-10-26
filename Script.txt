// ==UserScript==
// @name         Renombrar capturas pegadas v3
// @match        *://*/*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(() => {
  const renombrar = (f) => {
    if (!f || !f.type.startsWith('image/') || !/^imagen?\.\w+$/i.test(f.name)) return f;
    const ext = f.type.split('/')[1];
    return new File([f], `captura_${Date.now()}_${Math.floor(Math.random() * 1000)}.${ext}`,
      { type: f.type, lastModified: Date.now() });
  };

  // Cuando la página pide el archivo de un item del portapapeles
  const origGet = DataTransferItem.prototype.getAsFile;
  DataTransferItem.prototype.getAsFile = function () {
    return renombrar(origGet.call(this));
  };

  // Cuando la página lee clipboardData.files
  const desc = Object.getOwnPropertyDescriptor(DataTransfer.prototype, 'files');
  Object.defineProperty(DataTransfer.prototype, 'files', {
    configurable: true,
    get() {
      const lista = desc.get.call(this);
      if (!lista || !lista.length) return lista;
      const dt = new DataTransfer();
      for (const f of lista) dt.items.add(renombrar(f));
      return desc.get.call(dt);
    }
  });
})();
