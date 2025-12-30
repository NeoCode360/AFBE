import React, { useState, useEffect, useRef } from 'react';
import { 
  Upload, FileCode, Settings, CheckCircle, ArrowRight, ArrowLeft, 
  Database, Shield, Repeat, Box, Download, Server, 
  AlertCircle, Terminal as TerminalIcon, Layers, ToggleLeft, ToggleRight, XCircle, AlertTriangle 
} from 'lucide-react';

/**
 * =================================================================================
 * 📁 src/index.css
 * INSTRUCTIONS: In your local project, paste this content into src/index.css
 * =================================================================================
 */
const cssStyles = `
  :root {
    --bg-page: #0f172a;
    --bg-panel: #1e293b;
    --border-color: #334155;
    --text-main: #f8fafc;
    --text-muted: #94a3b8;
    --primary: #3b82f6;
    --primary-hover: #2563eb;
    --success: #22c55e;
    --success-bg: rgba(34, 197, 94, 0.1);
    --error: #ef4444;
    --error-bg: rgba(239, 68, 68, 0.1);
    --mono-font: 'Menlo', 'Monaco', 'Courier New', monospace;
  }

  * { box-sizing: border-box; }

  html, body { 
    margin: 0; 
    padding: 0;
    width: 100%;
    min-height: 100vh;
    font-family: ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; 
    background-color: var(--bg-page) !important; 
    color: var(--text-main); 
  }

  h1, h2, h3, h4, h5, h6 { color: var(--text-main); margin: 0; font-weight: 700; line-height: 1.2; }
  p { color: var(--text-muted); margin: 0; }
  
  /* Main Container */
  .af-app { 
    min-height: 100vh; 
    padding: 2rem; 
    display: flex; 
    justify-content: center; 
    width: 100%;
    background-color: var(--bg-page);
  }
  
  @media (max-width: 640px) {
    .af-app { padding: 1rem; }
  }
  
  .af-container { width: 100%; max-width: 1280px; display: flex; flex-direction: column; gap: 2rem; }

  /* Header */
  .af-header { 
    display: flex; 
    justify-content: space-between; 
    align-items: center; 
    border-bottom: 1px solid var(--border-color); 
    padding-bottom: 1.5rem; 
    flex-wrap: wrap; 
    gap: 1rem; 
  }
  
  @media (max-width: 640px) {
    .af-header { flex-direction: column; align-items: flex-start; gap: 1.5rem; }
    .af-controls { width: 100%; justify-content: space-between; }
  }

  .af-brand { display: flex; align-items: center; gap: 1rem; }
  .af-logo { background: var(--primary); padding: 0.5rem; border-radius: 0.5rem; color: white; box-shadow: 0 0 15px rgba(59, 130, 246, 0.3); flex-shrink: 0; }
  .af-title h1 { font-size: 1.5rem; letter-spacing: -0.025em; }
  .af-title p { font-size: 0.75rem; color: var(--text-muted); text-transform: uppercase; letter-spacing: 0.05em; font-weight: 600; margin-top: 0.25rem; }
  
  .af-controls { display: flex; align-items: center; gap: 1rem; }
  .af-mode-switch { display: flex; align-items: center; gap: 0.5rem; font-size: 0.75rem; color: var(--text-muted); cursor: pointer; border: 1px solid var(--border-color); padding: 0.25rem 0.75rem; border-radius: 99px; transition: 0.2s; white-space: nowrap; }
  .af-mode-switch:hover { background: rgba(255,255,255,0.05); }
  .af-mode-switch.sim { color: var(--success); border-color: var(--success); }
  .af-mode-switch.disabled { opacity: 0.5; cursor: not-allowed; pointer-events: none; border-color: var(--border-color); }
  
  .af-status { display: flex; align-items: center; gap: 0.5rem; font-family: monospace; font-size: 0.75rem; background: var(--bg-panel); padding: 0.5rem 1rem; border-radius: 99px; border: 1px solid var(--border-color); color: var(--text-muted); white-space: nowrap; }
  .af-dot { width: 8px; height: 8px; border-radius: 50%; background: #475569; }
  .af-dot.active { background: var(--success); box-shadow: 0 0 8px var(--success); }

  /* Steps Navigation Path */
  .af-steps { 
    display: flex; 
    justify-content: space-between; 
    align-items: center; 
    padding: 0 1rem; 
    position: relative; 
    margin-bottom: 1rem;
    overflow-x: auto;
    scrollbar-width: none;
    -ms-overflow-style: none;
  }
  .af-steps::-webkit-scrollbar { display: none; }

  .af-steps::before { 
    content: ''; 
    position: absolute; 
    top: 50%; 
    left: 2rem; 
    right: 2rem; 
    height: 2px; 
    background: var(--border-color); 
    z-index: 0; 
    transform: translateY(-50%); 
    min-width: 300px;
  }
  
  .af-step { 
    display: flex; 
    align-items: center; 
    gap: 0.75rem; 
    color: var(--text-muted); 
    position: relative; 
    z-index: 10; 
    background: var(--bg-page); 
    padding: 0 0.5rem; 
  }
  .af-step.active { color: var(--primary); }
  .af-step.done { color: var(--success); }
  
  .af-step-circle { 
    width: 2.5rem; 
    height: 2.5rem; 
    border-radius: 50%; 
    border: 2px solid var(--border-color); 
    display: flex; 
    align-items: center; 
    justify-content: center; 
    font-weight: bold; 
    font-size: 0.9rem; 
    transition: 0.3s; 
    background: var(--bg-page); 
    color: var(--text-muted);
    flex-shrink: 0;
  }
  .af-step.active .af-step-circle { background: rgba(59, 130, 246, 0.1); border-color: var(--primary); color: var(--primary); box-shadow: 0 0 0 4px rgba(59, 130, 246, 0.1); }
  .af-step.done .af-step-circle { background: var(--success-bg); border-color: var(--success); color: var(--success); }
  
  .af-step-label { display: none; font-weight: 600; font-size: 0.875rem; white-space: nowrap; }
  @media (min-width: 768px) { .af-step-label { display: inline; } }

  /* Main Layout */
  .af-grid { display: grid; grid-template-columns: 2.5fr 1fr; gap: 2rem; align-items: start; }
  @media (max-width: 1024px) { .af-grid { grid-template-columns: 1fr; gap: 2rem; } }

  /* Panels */
  .af-panel { background: var(--bg-panel); border: 1px solid var(--border-color); border-radius: 1rem; overflow: hidden; display: flex; flex-direction: column; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.3); }
  .af-content { padding: 2.5rem; flex: 1; }
  .af-footer { padding: 1.5rem 2.5rem; border-top: 1px solid var(--border-color); background: rgba(15, 23, 42, 0.5); display: flex; justify-content: space-between; align-items: center; }

  @media (max-width: 640px) {
    .af-content { padding: 1.5rem; }
    .af-footer { padding: 1.5rem; }
  }

  /* Error Block */
  .af-error { margin-bottom: 1.5rem; padding: 1rem; border-radius: 0.5rem; background: var(--error-bg); border: 1px solid var(--error); color: #fca5a5; display: flex; align-items: start; gap: 0.75rem; animation: fadeIn 0.3s ease-out; }
  .af-error strong { color: white; display: block; margin-bottom: 0.25rem; }

  /* Animations */
  @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
  .animate-enter { animation: fadeIn 0.4s ease-out forwards; }
  @keyframes spin { to { transform: rotate(360deg); } }
  .spinner { width: 1.5rem; height: 1.5rem; border: 3px solid var(--border-color); border-top-color: white; border-radius: 50%; animation: spin 1s linear infinite; }

  /* Inputs & Forms */
  .af-group { margin-bottom: 1.5rem; }
  .af-label { display: block; margin-bottom: 0.5rem; font-size: 0.875rem; color: var(--text-muted); font-weight: 500; }
  .af-input { width: 100%; background: var(--bg-page); border: 1px solid var(--border-color); padding: 0.875rem; border-radius: 0.5rem; color: white; outline: none; transition: 0.2s; font-family: inherit; font-size: 0.95rem; }
  .af-input:focus { border-color: var(--primary); box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.2); }
  
  .af-row { display: grid; grid-template-columns: 1fr 1fr; gap: 1.5rem; }
  @media (max-width: 640px) { .af-row { grid-template-columns: 1fr; gap: 1rem; } }

  /* Upload Cards */
  .af-upload { border: 2px dashed var(--border-color); padding: 2.5rem; border-radius: 1rem; text-align: center; transition: 0.2s; cursor: pointer; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100%; min-height: 250px; }
  .af-upload:hover { border-color: var(--primary); background: rgba(59, 130, 246, 0.05); }
  .af-upload.active { border-color: var(--success); background: var(--success-bg); }
  .af-upload-icon { margin-bottom: 1rem; color: #64748b; transition: 0.2s; }
  .af-upload.active .af-upload-icon { color: var(--success); }
  .af-upload h4 { color: var(--text-main); margin: 0 0 0.5rem 0; font-size: 1.1rem; font-weight: 600; }
  .af-upload p { color: var(--text-muted); font-size: 0.85rem; margin: 0 0 1rem 0; }

  /* Feature Cards */
  .af-feature { padding: 1.25rem; background: var(--bg-page); border: 1px solid var(--border-color); border-radius: 0.75rem; cursor: pointer; transition: 0.2s; display: flex; align-items: start; gap: 1rem; position: relative; }
  .af-feature:hover { border-color: #475569; transform: translateY(-2px); }
  .af-feature.selected { border-color: var(--primary); background: rgba(59, 130, 246, 0.05); }
  .af-feature-icon { color: var(--text-muted); flex-shrink: 0; }
  .af-feature.selected .af-feature-icon { color: var(--primary); }
  .af-feature h4 { margin: 0 0 0.25rem; font-weight: 600; color: white; }
  .af-feature p { margin: 0; font-size: 0.8rem; color: var(--text-muted); line-height: 1.4; }

  /* Build Screen */
  .af-build-screen { text-align: center; padding: 4rem 0; }
  .af-build-screen h2 { color: var(--text-main); font-size: 1.5rem; margin-bottom: 0.5rem; }
  .af-build-screen p { color: var(--text-muted); }
  .af-build-status { margin-top: 1rem; font-family: var(--mono-font); font-size: 0.8rem; color: var(--primary); background: rgba(59, 130, 246, 0.1); display: inline-block; padding: 0.5rem 1rem; border-radius: 99px; }

  /* Error Panel */
  .af-error-panel { border: 1px solid var(--error); background: rgba(239, 68, 68, 0.05); border-radius: 1rem; padding: 2rem; text-align: center; animation: fadeIn 0.3s ease-out; }
  .af-error-details { background: rgba(0, 0, 0, 0.3); border: 1px solid var(--border-color); padding: 1.5rem; border-radius: 0.5rem; margin: 1.5rem auto; max-width: 600px; text-align: left; overflow-x: auto; color: #fca5a5; font-family: var(--mono-font); font-size: 0.85rem; }

  /* Terminal */
  .af-terminal { font-family: var(--mono-font); font-size: 0.8rem; background: #0b0f19; padding: 1rem; border-radius: 0.75rem; height: 450px; overflow-y: auto; color: #a5f3fc; border: 1px solid var(--border-color); box-shadow: inset 0 0 20px rgba(0,0,0,0.5); }
  .log-line { margin-bottom: 0.5rem; display: flex; gap: 0.75rem; line-height: 1.4; }
  .log-ts { color: #475569; user-select: none; min-width: 80px; flex-shrink: 0; }
  .log-msg { color: #e2e8f0; word-break: break-word; }
  .log-msg.highlight { color: var(--success); font-weight: bold; }
  .log-msg.info { color: var(--primary); }
  .log-msg.error { color: var(--error); font-weight: bold; }

  /* Buttons */
  .btn { display: inline-flex; align-items: center; gap: 0.5rem; padding: 0.75rem 1.5rem; border-radius: 0.5rem; font-weight: 600; border: none; cursor: pointer; transition: 0.2s; font-size: 0.95rem; white-space: nowrap; }
  .btn-primary { background: var(--primary); color: white; box-shadow: 0 4px 6px -1px rgba(37, 99, 235, 0.3); }
  .btn-primary:hover:not(:disabled) { background: var(--primary-hover); transform: translateY(-1px); }
  .btn-primary:disabled { opacity: 0.6; cursor: not-allowed; filter: grayscale(1); }
  .btn-secondary { background: var(--border-color); color: white; }
  .btn-secondary:hover { background: #475569; }
  .btn-nav { background: transparent; color: var(--text-muted); }
  .btn-nav:hover { color: white; background: rgba(255,255,255,0.05); }

  /* Info Box */
  .af-info { margin-top: 1.5rem; padding: 1.5rem; background: rgba(59, 130, 246, 0.05); border: 1px solid rgba(59, 130, 246, 0.2); border-radius: 0.75rem; }
  .af-info h3 { margin: 0 0 0.5rem; font-size: 0.9rem; font-weight: 700; color: var(--primary); display: flex; align-items: center; gap: 0.5rem; }
  .af-info p { margin: 0; font-size: 0.85rem; color: var(--text-muted); line-height: 1.6; }
`;

/**
 * =================================================================================
 * 📁 src/types/index.ts
 * =================================================================================
 */

export interface ProjectMeta {
  name: string;
  groupId: string;
  artifactId: string;
  version: string;
  description: string;
}

export interface FeatureFlags {
  idempotency: boolean;
  validation: boolean;
  entitlements: boolean;
  auditLogging: boolean;
}

export interface LogEntry {
  msg: string;
  type?: 'info' | 'highlight' | 'error';
}

/**
 * =================================================================================
 * 📁 src/services/apiFactory.ts
 * =================================================================================
 */

const API_BASE = "http://localhost:8080/api/projects";

export const FactoryAPI = {
  createProject: async (meta: ProjectMeta) => {
    // Mock simulation for preview
    return { id: "PROJ-" + Math.floor(Math.random() * 10000) };
  },
  uploadSwagger: async (id: string, file: File) => { return; },
  uploadJar: async (id: string, file: File) => { return; },
  updateConfig: async (id: string, features: FeatureFlags) => { return; },
  generate: async (id: string) => { return; },
  getDownloadLink: (id: string) => `${API_BASE}/${id}/download`
};

/**
 * =================================================================================
 * 📁 src/components/ui/StepIcon.tsx
 * =================================================================================
 */
export const StepIcon = ({ num, current, label }: { num: number; current: number; label: string }) => (
  <div className={`af-step ${current === num ? 'active' : current > num ? 'done' : ''}`}>
    <div className="af-step-circle">
      {current > num ? <CheckCircle size={20}/> : num}
    </div>
    <span className="af-step-label">{label}</span>
  </div>
);

/**
 * =================================================================================
 * 📁 src/components/layout/Header.tsx
 * =================================================================================
 */
export const Header = ({ isSimulationMode, toggleSimulation, step, projectId }: any) => (
  <header className="af-header">
    <div className="af-brand">
      <div className="af-logo"><Server size={24}/></div>
      <div className="af-title">
        <h1>API Factory</h1>
        <p>Spring Boot Microservice Generator</p>
      </div>
    </div>
    <div className="af-controls">
      <div 
        className={`af-mode-switch ${isSimulationMode ? 'sim' : ''} ${step > 1 ? 'disabled' : ''}`} 
        onClick={() => step === 1 && toggleSimulation()}
      >
        {isSimulationMode ? <ToggleRight size={18}/> : <ToggleLeft size={18}/>}
        <span>{isSimulationMode ? 'SIMULATION MODE' : 'REAL BACKEND'}</span>
      </div>
      <div className="af-status">
        <div className={`af-dot ${projectId ? 'active' : ''}`}/>
        {projectId ? `ID: ${projectId.split('-')[1]}` : 'READY'}
      </div>
    </div>
  </header>
);

/**
 * =================================================================================
 * 📁 src/components/layout/Terminal.tsx
 * =================================================================================
 */
export const Terminal = ({ logs, isLoading }: { logs: LogEntry[]; isLoading: boolean }) => {
  const terminalRef = useRef<HTMLDivElement>(null);
  useEffect(() => {
    if (terminalRef.current) terminalRef.current.scrollTop = terminalRef.current.scrollHeight;
  }, [logs]);

  return (
    <div className="af-panel" style={{height: '100%'}}>
      <div style={{padding: '1rem', borderBottom: '1px solid var(--border-color)', display: 'flex', alignItems: 'center', gap: '0.5rem', color: '#94a3b8', fontSize: '0.8rem', fontWeight: 'bold'}}>
        <TerminalIcon size={16}/> GENERATOR LOGS
      </div>
      <div ref={terminalRef} className="af-terminal">
        {logs.length === 0 ? <span style={{color: '#475569'}}>Waiting for activity...</span> : logs.map((l, i) => (
          <div key={i} className="log-line">
            <span className="log-ts">{l.msg.match(/\d+:\d+:\d+/)?.[0] || '..:..:..'}</span>
            <span className={`log-msg ${l.type || ''}`}>{l.msg.replace(/^\d+:\d+:\d+\s/, '')}</span>
          </div>
        ))}
        {isLoading && <span className="animate-pulse text-blue-400">_</span>}
      </div>
    </div>
  );
};

/**
 * =================================================================================
 * 📁 src/components/wizard/DefinitionStep.tsx
 * =================================================================================
 */
export const DefinitionStep = ({ meta, setMeta }: { meta: ProjectMeta; setMeta: any }) => (
  <div className="animate-enter">
    <h2 className="text-xl font-bold mb-6 flex gap-2 text-white"><Box className="text-blue-500"/> Project Definition</h2>
    <div className="af-row">
      <div className="af-group">
        <label className="af-label">Group ID</label>
        <input className="af-input" value={meta.groupId} onChange={e => setMeta({...meta, groupId: e.target.value})} />
      </div>
      <div className="af-group">
        <label className="af-label">Artifact ID</label>
        <input className="af-input" value={meta.artifactId} onChange={e => setMeta({...meta, artifactId: e.target.value})} />
      </div>
    </div>
    <div className="af-row">
      <div className="af-group">
        <label className="af-label">Version</label>
        <input className="af-input" value={meta.version} onChange={e => setMeta({...meta, version: e.target.value})} />
      </div>
      <div className="af-group">
        <label className="af-label">App Name</label>
        <input className="af-input" value={meta.name} onChange={e => setMeta({...meta, name: e.target.value})} />
      </div>
    </div>
    <div className="af-group">
      <label className="af-label">Description</label>
      <textarea className="af-input" style={{height: '80px', resize: 'none'}} value={meta.description} onChange={e => setMeta({...meta, description: e.target.value})} />
    </div>
  </div>
);

/**
 * =================================================================================
 * 📁 src/components/wizard/AssetsStep.tsx
 * =================================================================================
 */
export const AssetsStep = ({ swaggerFile, setSwaggerFile, jarFile, setJarFile }: any) => (
  <div className="animate-enter">
    <h2 className="text-xl font-bold mb-6 flex gap-2 text-white"><Upload className="text-blue-500"/> Upload Contracts</h2>
    <div className="af-row" style={{height: '300px'}}>
      <div className={`af-upload ${swaggerFile ? 'active' : ''}`}>
        <FileCode size={48} className={`af-upload-icon ${swaggerFile ? 'active' : ''}`} />
        <h4>OpenAPI Spec</h4>
        <p>Required (.json/.yaml)</p>
        <label className="btn btn-secondary text-xs">
          <input type="file" hidden onChange={e => setSwaggerFile(e.target.files?.[0] || null)}/>
          {swaggerFile ? 'Replace File' : 'Select File'}
        </label>
        {swaggerFile && <div className="text-xs text-green-400 mt-2 font-mono">{swaggerFile.name}</div>}
      </div>
      <div className={`af-upload ${jarFile ? 'active' : ''}`}>
        <Box size={48} className={`af-upload-icon ${jarFile ? 'active' : ''}`} />
        <h4>Business Logic</h4>
        <p>Optional (.jar)</p>
        <label className="btn btn-secondary text-xs">
          <input type="file" hidden onChange={e => setJarFile(e.target.files?.[0] || null)}/>
          {jarFile ? 'Replace File' : 'Select File'}
        </label>
        {jarFile && <div className="text-xs text-green-400 mt-2 font-mono">{jarFile.name}</div>}
      </div>
    </div>
  </div>
);

/**
 * =================================================================================
 * 📁 src/components/wizard/FeaturesStep.tsx
 * =================================================================================
 */
export const FeaturesStep = ({ features, setFeatures }: any) => (
  <div className="animate-enter">
    <h2 className="text-xl font-bold mb-6 flex gap-2 text-white"><Settings className="text-blue-500"/> Middleware Configuration</h2>
    <div className="af-row">
      {[
        {k: 'idempotency', t: 'Idempotency', d: 'Redis-backed deduplication.', i: <Repeat/>},
        {k: 'entitlements', t: 'Entitlements', d: 'RBAC checks injected via AOP.', i: <Shield/>},
        {k: 'auditLogging', t: 'Audit Log', d: 'Async payload logging to ELK.', i: <Database/>},
        {k: 'validation', t: 'Validation', d: 'JSR-303 Schema enforcement.', i: <CheckCircle/>}
      ].map(f => (
        <div key={f.k} className={`af-feature ${features[f.k as keyof FeatureFlags] ? 'selected' : ''}`} onClick={() => setFeatures({...features, [f.k as keyof FeatureFlags]: !features[f.k as keyof FeatureFlags]})}>
          <div className="af-feature-icon">{f.i}</div>
          <div><h4>{f.t}</h4><p>{f.d}</p></div>
          {features[f.k as keyof FeatureFlags] && <CheckCircle size={18} className="text-blue-500" style={{position:'absolute', top:'1rem', right:'1rem', color:'#3b82f6'}}/>}
        </div>
      ))}
    </div>
  </div>
);

/**
 * =================================================================================
 * 📁 src/components/wizard/BuildStep.tsx
 * =================================================================================
 */
export const BuildStep = ({ error, buildStatus, onRetry, onReupload, onEditConfig }: any) => (
  <div className="animate-enter af-build-screen">
    {error ? (
      <div className="af-error-panel">
          <div style={{color: '#ef4444', marginBottom: '1rem', display: 'flex', justifyContent: 'center'}}><AlertTriangle size={64} /></div>
          <h2 className="text-2xl font-bold mb-2" style={{color: '#ef4444'}}>Manufacturing Failed</h2>
          <div className="af-error-details">
              <p style={{color: '#fca5a5', fontFamily:'monospace', marginBottom:'0.5rem'}}>{error}</p>
              <p className="text-xs text-slate-500">Trace: {buildStatus || 'Init'}</p>
          </div>
          <div style={{display: 'flex', gap: '1rem', marginTop: '2rem', justifyContent: 'center', flexWrap: 'wrap'}}>
              <button className="btn btn-secondary" onClick={onReupload}>Re-upload Assets</button>
              <button className="btn btn-secondary" onClick={onEditConfig}>Edit Config</button>
              <button className="btn btn-primary" onClick={onRetry}>Retry Build</button>
          </div>
      </div>
    ) : (
      <>
          <div className="spinner" style={{width: '4rem', height: '4rem', margin: '0 auto 2rem'}}></div>
          <h2 className="text-2xl font-bold mb-2" style={{color: 'white'}}>Manufacturing API</h2>
          <p className="text-slate-400">Generates Multi-Module Maven Project...</p>
          {buildStatus && <div className="af-build-status">{buildStatus}</div>}
      </>
    )}
  </div>
);

/**
 * =================================================================================
 * 📁 src/components/wizard/ResultStep.tsx
 * =================================================================================
 */
export const ResultStep = ({ meta, onDownload, onReset }: any) => (
  <div className="animate-enter" style={{textAlign: 'center', padding: '2rem 0'}}>
    <div style={{display: 'inline-flex', padding: '1.5rem', borderRadius: '50%', background: 'rgba(34, 197, 94, 0.1)', color: '#22c55e', marginBottom: '1.5rem'}}>
      <CheckCircle size={64}/>
    </div>
    <h2 className="text-3xl font-bold mb-4 text-white">Project Generated</h2>
    <div style={{background: '#0f172a', display: 'inline-block', padding: '1rem 2rem', borderRadius: '0.5rem', border: '1px solid #334155', marginBottom: '2rem', textAlign: 'left'}}>
      <div className="text-slate-400 text-xs uppercase mb-2">Structure</div>
      <div className="font-mono text-sm text-blue-300">
        <div>📦 {meta.artifactId}-root</div>
        <div className="pl-4">├── 📦 {meta.artifactId}-contract <span className="text-slate-500">(OpenAPI)</span></div>
        <div className="pl-4">└── 📦 {meta.artifactId}-service <span className="text-slate-500">(Spring Boot)</span></div>
      </div>
    </div>
    
    <div style={{display: 'flex', gap: '1rem', justifyContent: 'center'}}>
      <button className="btn btn-primary" onClick={onDownload}><Download size={18}/> Download ZIP</button>
      <button className="btn btn-secondary" onClick={onReset}>Create Another</button>
    </div>
  </div>
);

/**
 * =================================================================================
 * 📁 src/App.tsx
 * =================================================================================
 */

export default function App() {
  const [step, setStep] = useState(1);
  const [isLoading, setIsLoading] = useState(false);
  const [projectId, setProjectId] = useState<string | null>(null);
  const [logs, setLogs] = useState<LogEntry[]>([]);
  const [error, setError] = useState<string | null>(null);
  const [isSimulationMode, setIsSimulationMode] = useState(true);
  const [buildStatus, setBuildStatus] = useState<string>("");
  
  const [meta, setMeta] = useState<ProjectMeta>({
    name: 'OrderProcessingAPI',
    groupId: 'com.enterprise.orders',
    artifactId: 'order-service-api',
    version: '1.0.0-SNAPSHOT',
    description: 'Auto-generated API for Order Management System'
  });

  const [swaggerFile, setSwaggerFile] = useState<File | null>(null);
  const [jarFile, setJarFile] = useState<File | null>(null);
  
  const [features, setFeatures] = useState<FeatureFlags>({
    idempotency: false,
    validation: true,
    entitlements: false,
    auditLogging: false,
  });

  const addLog = (msg: string, type?: 'info' | 'highlight' | 'error') => {
    setLogs(prev => [...prev, { msg: `[${new Date().toLocaleTimeString()}] ${msg}`, type }]);
    if (type === 'highlight' || type === 'info') {
        const cleanMsg = msg.replace(/^\[.*?\]\s*/, '');
        setBuildStatus(cleanMsg);
    }
  };

  const handleCreateDraft = async () => {
    setIsLoading(true);
    setError(null);
    addLog("POST /api/projects - Initializing Session...");
    
    // Simulate API delay
    setTimeout(() => {
        const id = "PROJ-" + Math.floor(Math.random() * 10000);
        setProjectId(id);
        addLog(`[SIM] Session Created: ${id}`, 'highlight');
        setIsLoading(false);
        setStep(2);
    }, 800);
  };

  const handleUpload = async () => {
    if (!projectId) return;
    setIsLoading(true);
    setError(null);
    addLog(`Uploading Assets for ${projectId}...`);
    setTimeout(() => {
        addLog("[SIM] OpenAPI Spec validated.", 'info');
        addLog("[SIM] Assets stored in In-Memory DB.");
        setIsLoading(false);
        setStep(3);
    }, 1200);
  };

  const handleConfig = async () => {
    if (!projectId) return;
    setIsLoading(true);
    setError(null);
    addLog("Saving Configuration...");
    setTimeout(() => {
        addLog("[SIM] Configuration persisted.", 'info');
        setIsLoading(false);
        setStep(4);
    }, 800);
  }

  const generateCalled = useRef(false);
  useEffect(() => {
      if (step === 4 && !generateCalled.current && !isLoading && !error) {
          generateCalled.current = true;
          handleGenerate();
      }
      if (step !== 4) {
          generateCalled.current = false;
      }
  }, [step]);

  const handleGenerate = async () => {
    if (!projectId) return;
    setIsLoading(true);
    setError(null); 
    
    addLog(`[SIM] Starting In-Memory Build...`, 'highlight');
    const willFail = Math.random() < 0.3; // 30% failure chance
    
    if (willFail) {
      const failSequence = [
        { t: 500, msg: "Initializing Generator Engine..." },
        { t: 1500, msg: "Scaffolding Multi-Module Maven Project...", type: 'info' },
        { t: 2500, msg: "Creating module: 'contract' (OpenAPI Generator)" },
        { t: 3000, msg: "Error: Mustache Template Syntax Error in 'contract-pom.xml'", type: 'error' },
        { t: 3500, msg: "Build Failed. Rolling back changes.", type: 'error' }
      ];
      failSequence.forEach(({ t, msg, type }) => {
        setTimeout(() => {
           addLog(msg, type as any);
           if (t === 3500) {
             setIsLoading(false);
             setError("Template Rendering Exception: Missing required property 'groupId'");
             setBuildStatus("Failed during Contract Module generation");
           }
        }, t);
      });
    } else {
      const sequence = [
          { t: 500, msg: "Initializing Generator Engine..." },
          { t: 1500, msg: "Scaffolding Multi-Module Maven Project...", type: 'info' },
          { t: 2500, msg: "Creating module: 'root' (Aggregator POM)" },
          { t: 3000, msg: "Creating module: 'contract' (OpenAPI Generator)" },
          { t: 3500, msg: "Creating module: 'service' (Spring Boot App)" },
          { t: 4000, msg: "Injecting Delegate Impl..." },
          { t: 5000, msg: "Zipping artifacts...", type: 'highlight' }
      ];
      sequence.forEach(({ t, msg, type }) => {
          setTimeout(() => {
            addLog(msg, type as any);
            if (t === 5000) {
                setIsLoading(false);
                setStep(5);
            }
          }, t);
      });
    }
  };

  const handleDownload = () => {
    if (isSimulationMode) {
      alert("This is a simulation. In real mode, this downloads the ZIP.");
    } else if (projectId) {
      window.location.href = FactoryAPI.getDownloadLink(projectId);
    }
  };

  const handleReset = () => {
    setStep(1);
    setProjectId(null);
    setSwaggerFile(null);
    setJarFile(null);
    setLogs([]);
    setError(null);
    setBuildStatus("");
    generateCalled.current = false;
  };

  return (
    <div className="af-app">
      <style>{cssStyles}</style>
      <div className="af-container">
        
        <Header 
          isSimulationMode={isSimulationMode}
          toggleSimulation={() => setIsSimulationMode(!isSimulationMode)}
          step={step}
          projectId={projectId}
        />

        <div className="af-steps">
          <StepIcon num={1} current={step} label="Definition" />
          <StepIcon num={2} current={step} label="Assets" />
          <StepIcon num={3} current={step} label="Features" />
          <StepIcon num={4} current={step} label="Manufacture" />
          <StepIcon num={5} current={step} label="Artifact" />
        </div>

        {error && step < 4 && (
          <div className="af-error">
            <AlertCircle size={24} />
            <div>
              <strong>Process Failed</strong>
              <span>{error}</span>
            </div>
            <button onClick={() => setError(null)} style={{marginLeft:'auto', background:'none', border:'none', color:'inherit', cursor:'pointer'}}><XCircle size={18}/></button>
          </div>
        )}

        <div className="af-grid">
          <div className="af-panel">
            <div className="af-content">
              
              {/* Step 1: Definition */}
              {step === 1 && <DefinitionStep meta={meta} setMeta={setMeta} />}

              {/* Step 2: Assets */}
              {step === 2 && <AssetsStep swaggerFile={swaggerFile} setSwaggerFile={setSwaggerFile} jarFile={jarFile} setJarFile={setJarFile} />}

              {/* Step 3: Features */}
              {step === 3 && <FeaturesStep features={features} setFeatures={setFeatures} />}

              {/* Step 4: Build */}
              {step === 4 && (
                <BuildStep 
                  error={error} 
                  buildStatus={buildStatus} 
                  onRetry={() => { setError(null); handleGenerate(); }}
                  onReupload={() => { setError(null); setStep(2); }}
                  onEditConfig={() => { setError(null); setStep(3); }}
                />
              )}

              {/* Step 5: Result */}
              {step === 5 && <ResultStep meta={meta} onDownload={handleDownload} onReset={handleReset} />}
            </div>

            {/* Footer */}
            {step < 4 && (
              <div className="af-footer" style={{justifyContent: step === 1 ? 'flex-end' : 'space-between'}}>
                {step > 1 && (
                  <button className="btn btn-nav" disabled={step === 1} onClick={() => setStep(step - 1)}>
                    <ArrowLeft size={16}/> Back
                  </button>
                )}
                <button 
                  className="btn btn-primary" 
                  disabled={isLoading || (step === 2 && !swaggerFile)}
                  onClick={step === 1 ? handleCreateDraft : step === 2 ? handleUpload : handleConfig}
                >
                  {isLoading ? <><div className="spinner" style={{width:'1em', height:'1em', borderTopColor:'white', borderRightColor:'white', borderBottomColor:'white'}}></div> Processing...</> : <>{step === 3 ? 'Generate API' : 'Next Step'} <ArrowRight size={16}/></>}
                </button>
              </div>
            )}
          </div>

          {/* Right Panel */}
          <div style={{display: 'flex', flexDirection: 'column', gap: '1.5rem'}}>
            <Terminal logs={logs} isLoading={isLoading} />
            <div className="af-info">
              <h3><Layers size={16}/> Architecture</h3>
              <p>
                <strong>{isSimulationMode ? 'SIMULATION' : 'LIVE'} Mode:</strong> The Factory scaffolds a 3-module Maven structure (`root`, `contract`, `service`) into a ZIP file.
              </p>
            </div>
          </div>

        </div>
      </div>
    </div>
  );
}