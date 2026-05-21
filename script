// ==UserScript==
// @name         Project Bunnycloak (Safe Release)
// @namespace    https://yourdomain.com
// @version      2.1.0
// @description  Safer Bunnycloak build with high-signal location redaction, protected editors/inputs, and reduced false positives.
// @author       Flatline
// @match        *://*/*
// @run-at       document-end
// @grant        none
// ==/UserScript==
 
(() => {
    'use strict';
 
    const CFG = {
        enabled: true,
        token: '[REDACTED]',
        debug: false,
        maxLogs: 200,
 
        // Safety defaults
        skipEditable: true,
        skipTags: new Set([
            'SCRIPT',
            'STYLE',
            'NOSCRIPT',
            'TEXTAREA',
            'INPUT',
            'CODE',
            'PRE',
            'KBD',
            'SAMP',
            'SELECT',
            'OPTION'
        ]),
 
        // Prevent touching Bunnycloak's own UI
        uiRootIds: new Set([
            'bunnycloak-watermark',
            'bunnycloak-gear',
            'bunnycloak-menu',
            'bunnycloak-log',
            'bunnycloak-style'
        ]),
 
        mutationDebounceMs: 120,
 
        // Hotkeys
        toggleHotkey: { altKey: true, key: 'b' }, // Alt+B
        menuHotkey: { altKey: true, key: 'm' },   // Alt+M
 
        // Safer, high-signal wordlist only
        // No short abbreviations, no "from", no "ip", no weak shorthand like lat/long
        locationWords: [
            // State / country / city names you explicitly care about
            'illinois',
            'california',
            'texas',
            'new york',
            'japan',
            'kyoto',
            'tokyo',
 
            // Privacy / location leak phrases
            'zip code',
            'zipcode',
            'postal code',
            'postcode',
            'area code',
            'your location',
            'current location',
            'based in',
            'hometown',
            'city of',
 
            // Higher-signal technical leak terms
            'geoip',
            'geolocation',
            'gps',
            'latitude',
            'longitude',
            'timezone',
            'time zone',
            'coordinates',
            'coordinate location'
        ]
    };
 
    const settings = {
        watermarkMode: localStorage.getItem('bunnycloak-watermark-mode') || 'visible',
        logVisible: localStorage.getItem('bunnycloak-log-visible') === 'true'
    };
 
    const LOGS = [];
 
    function log(type, msg, data) {
        if (!CFG.debug) return;
        LOGS.push({ t: Date.now(), type, msg, data });
        if (LOGS.length > CFG.maxLogs) LOGS.shift();
        console.log(`[Bunnycloak] ${type}: ${msg}`, data ?? '');
    }
 
    function escapeRegExp(str) {
        return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    }
 
    function normalizeTermList(terms) {
        return [...new Set(
            terms
                .filter(Boolean)
                .map(v => String(v).trim().toLowerCase())
                .filter(Boolean)
        )].sort((a, b) => b.length - a.length);
    }
 
    const allTerms = normalizeTermList(CFG.locationWords);
 
    const termPattern = allTerms
        .map(term => `\\b${escapeRegExp(term)}\\b`)
        .join('|');
 
    const TERMS_RE = new RegExp(termPattern, 'gi');
 
    function isBunnycloakUI(el) {
        let node = el;
        while (node) {
            if (node.id && CFG.uiRootIds.has(node.id)) return true;
            node = node.parentElement;
        }
        return false;
    }
 
    function isInsideSkippedElement(node) {
        let el = node.nodeType === Node.ELEMENT_NODE ? node : node.parentElement;
 
        while (el) {
            if (isBunnycloakUI(el)) return true;
            if (CFG.skipTags.has(el.tagName)) return true;
 
            if (CFG.skipEditable) {
                if (el.isContentEditable) return true;
 
                const role = el.getAttribute?.('role');
                if (role && role.toLowerCase() === 'textbox') return true;
            }
 
            el = el.parentElement;
        }
 
        return false;
    }
 
    function redactText(text) {
        if (!text) return { out: text, changed: false, hits: [] };
 
        TERMS_RE.lastIndex = 0;
        const hits = [];
        let match;
 
        while ((match = TERMS_RE.exec(text)) !== null) {
            hits.push(match[0]);
            if (hits.length > 50) break;
        }
 
        if (hits.length === 0) {
            return { out: text, changed: false, hits: [] };
        }
 
        TERMS_RE.lastIndex = 0;
        const out = text.replace(TERMS_RE, CFG.token);
 
        return {
            out,
            changed: out !== text,
            hits: [...new Set(hits)]
        };
    }
 
    function logRedaction(hits) {
        if (!settings.logVisible) return;
 
        const logBox = document.getElementById('bunnycloak-log');
        if (!logBox) return;
 
        const line = document.createElement('div');
        line.textContent = `Redacted: ${hits.slice(0, 5).join(', ')}`;
        logBox.appendChild(line);
 
        while (logBox.children.length > 20) {
            logBox.removeChild(logBox.firstChild);
        }
    }
 
    function scrubTextNode(textNode) {
        if (!CFG.enabled) return;
        if (!textNode || textNode.nodeType !== Node.TEXT_NODE) return;
        if (!textNode.textContent || !textNode.textContent.trim()) return;
        if (isInsideSkippedElement(textNode)) return;
 
        const before = textNode.textContent;
        const { out, changed, hits } = redactText(before);
 
        if (!changed) return;
 
        textNode.textContent = out;
        log('REDACT', 'Text node redacted', { hits });
        logRedaction(hits);
    }
 
    function scrubSubtree(root) {
        if (!CFG.enabled || !root) return;
 
        if (root.nodeType === Node.TEXT_NODE) {
            scrubTextNode(root);
            return;
        }
 
        if (root.nodeType === Node.ELEMENT_NODE) {
            if (isBunnycloakUI(root)) return;
            if (CFG.skipTags.has(root.tagName)) return;
            if (isInsideSkippedElement(root)) return;
        }
 
        const walker = document.createTreeWalker(
            root,
            NodeFilter.SHOW_TEXT,
            {
                acceptNode(node) {
                    if (!node.textContent || !node.textContent.trim()) {
                        return NodeFilter.FILTER_REJECT;
                    }
                    if (isInsideSkippedElement(node)) {
                        return NodeFilter.FILTER_REJECT;
                    }
                    return NodeFilter.FILTER_ACCEPT;
                }
            }
        );
 
        let node;
        while ((node = walker.nextNode())) {
            scrubTextNode(node);
        }
    }
 
    let pending = [];
    let flushTimer = null;
 
    function flushMutations() {
        const nodes = pending;
        pending = [];
        flushTimer = null;
 
        for (const node of nodes) {
            scrubSubtree(node);
        }
    }
 
    const observer = new MutationObserver((mutations) => {
        if (!CFG.enabled) return;
 
        for (const mutation of mutations) {
            if (mutation.type === 'childList' && mutation.addedNodes?.length) {
                mutation.addedNodes.forEach(node => {
                    if (node.nodeType === Node.ELEMENT_NODE || node.nodeType === Node.TEXT_NODE) {
                        pending.push(node);
                    }
                });
            } else if (mutation.type === 'characterData' && mutation.target) {
                pending.push(mutation.target);
            }
        }
 
        if (!flushTimer) {
            flushTimer = setTimeout(flushMutations, CFG.mutationDebounceMs);
        }
    });
 
    function applyWatermarkMode() {
        const watermark = document.getElementById('bunnycloak-watermark');
        if (!watermark) return;
 
        if (settings.watermarkMode === 'visible') {
            watermark.style.display = 'block';
            watermark.style.opacity = '1';
        } else if (settings.watermarkMode === 'transparent') {
            watermark.style.display = 'block';
            watermark.style.opacity = '0.1';
        } else {
            watermark.style.display = 'none';
        }
    }
 
    function toggleWatermark() {
        const modes = ['visible', 'transparent', 'hidden'];
        const currentIndex = modes.indexOf(settings.watermarkMode);
        settings.watermarkMode = modes[(currentIndex + 1) % modes.length];
        localStorage.setItem('bunnycloak-watermark-mode', settings.watermarkMode);
        applyWatermarkMode();
    }
 
    function toggleLog() {
        settings.logVisible = !settings.logVisible;
        localStorage.setItem('bunnycloak-log-visible', String(settings.logVisible));
 
        const logBox = document.getElementById('bunnycloak-log');
        if (logBox) {
            logBox.style.display = settings.logVisible ? 'block' : 'none';
        }
    }
 
    function toggleMenu() {
        const menu = document.getElementById('bunnycloak-menu');
        if (!menu) return;
        menu.style.display = menu.style.display === 'block' ? 'none' : 'block';
    }
 
    function initUI() {
        if (!document.head || !document.body) return;
 
        if (!document.getElementById('bunnycloak-style')) {
            const style = document.createElement('style');
            style.id = 'bunnycloak-style';
            style.textContent = `
                #bunnycloak-watermark {
                    position: fixed;
                    bottom: 10px;
                    right: 10px;
                    background: rgba(20, 20, 20, 0.8);
                    color: white;
                    font-family: monospace;
                    padding: 6px 12px;
                    font-size: 12px;
                    border-radius: 8px;
                    z-index: 2147483644;
                    pointer-events: none;
                    transition: opacity 0.3s ease;
                }
                #bunnycloak-gear {
                    position: fixed;
                    bottom: 10px;
                    right: 10px;
                    width: 28px;
                    height: 28px;
                    font-size: 16px;
                    background: #111;
                    color: #ff69b4;
                    border: 1px solid #333;
                    border-radius: 50%;
                    z-index: 2147483645;
                    cursor: pointer;
                    text-align: center;
                    line-height: 28px;
                    font-weight: bold;
                    font-family: monospace;
                    user-select: none;
                }
                #bunnycloak-menu {
                    position: fixed;
                    bottom: 50px;
                    right: 10px;
                    background: #111;
                    color: white;
                    padding: 8px;
                    font-size: 13px;
                    border: 1px solid #444;
                    border-radius: 6px;
                    z-index: 2147483645;
                    display: none;
                    font-family: monospace;
                    min-width: 180px;
                }
                #bunnycloak-menu button {
                    margin-top: 6px;
                    width: 100%;
                    font-family: monospace;
                    cursor: pointer;
                }
                #bunnycloak-log {
                    position: fixed;
                    bottom: 10px;
                    left: 10px;
                    background: rgba(0, 0, 0, 0.85);
                    color: white;
                    max-height: 150px;
                    max-width: 320px;
                    overflow-y: auto;
                    padding: 8px;
                    font-size: 11px;
                    font-family: monospace;
                    border-radius: 6px;
                    z-index: 2147483644;
                    display: ${settings.logVisible ? 'block' : 'none'};
                    white-space: pre-wrap;
                    word-break: break-word;
                }
            `;
            document.head.appendChild(style);
        }
 
        if (!document.getElementById('bunnycloak-watermark')) {
            const watermark = document.createElement('div');
            watermark.id = 'bunnycloak-watermark';
            watermark.textContent = '🐰 Cloaked by Flatline';
            document.body.appendChild(watermark);
        }
 
        if (!document.getElementById('bunnycloak-gear')) {
            const gear = document.createElement('div');
            gear.id = 'bunnycloak-gear';
            gear.textContent = '⚙️';
            gear.title = 'Bunnycloak Settings';
            gear.addEventListener('click', toggleMenu);
            document.body.appendChild(gear);
        }
 
        if (!document.getElementById('bunnycloak-menu')) {
            const menu = document.createElement('div');
            menu.id = 'bunnycloak-menu';
            menu.innerHTML = `
                <div><b>Bunnycloak Settings</b></div>
                <button type="button" id="toggle-watermark">Toggle Watermark</button>
                <button type="button" id="toggle-log">Toggle Redaction Log</button>
                <button type="button" id="toggle-cloak">Toggle Cloak</button>
            `;
            document.body.appendChild(menu);
 
            menu.querySelector('#toggle-watermark')?.addEventListener('click', toggleWatermark);
            menu.querySelector('#toggle-log')?.addEventListener('click', toggleLog);
            menu.querySelector('#toggle-cloak')?.addEventListener('click', () => {
                CFG.enabled = !CFG.enabled;
                log('TOGGLE', `enabled=${CFG.enabled}`);
                if (CFG.enabled) scrubSubtree(document.body);
            });
        }
 
        if (!document.getElementById('bunnycloak-log')) {
            const logBox = document.createElement('div');
            logBox.id = 'bunnycloak-log';
            document.body.appendChild(logBox);
        }
 
        applyWatermarkMode();
    }
 
    function handleHotkeys(e) {
        if (
            e.altKey === CFG.toggleHotkey.altKey &&
            (e.key || '').toLowerCase() === CFG.toggleHotkey.key
        ) {
            CFG.enabled = !CFG.enabled;
            log('TOGGLE', `enabled=${CFG.enabled}`);
            if (CFG.enabled) scrubSubtree(document.body);
        }
 
        if (
            e.altKey === CFG.menuHotkey.altKey &&
            (e.key || '').toLowerCase() === CFG.menuHotkey.key
        ) {
            toggleMenu();
        }
    }
 
    function start() {
        if (!document.body || !document.head) {
            setTimeout(start, 50);
            return;
        }
 
        initUI();
        scrubSubtree(document.body);
 
        observer.observe(document.body, {
            childList: true,
            subtree: true,
            characterData: true
        });
 
        window.addEventListener('keydown', handleHotkeys);
 
        log('INFO', 'Started Bunnycloak safe release.');
        log('INFO', 'Protected: inputs, editors, code blocks, and Bunnycloak UI.');
    }
 
    start();
})();
