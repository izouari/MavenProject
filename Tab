// sseClient.js
import { fetchEventSource } from '@microsoft/fetch-event-source';

let accessToken = null;

async function getAccessToken() { /* retourne le token actuel */ return accessToken; }
async function refreshToken() { /* rafraîchit et met à jour accessToken */ }

export async function openSSE(url, { signal, onmessage, onopen, onclose }) {
  const controller = new AbortController();
  const combinedSignal = signal
    ? new AbortController()
    : controller;

  // mini “interceptor” pour injecter le token et gérer 401
  let triedRefresh = false;

  await fetchEventSource(url, {
    signal: signal ?? controller.signal,
    headers: async () => ({
      Accept: 'text/event-stream',
      Authorization: `Bearer ${await getAccessToken()}`,
    }),
    openWhenHidden: true,
    onopen: async (resp) => {
      if (resp.ok) { onopen?.(resp); return; }
      if (resp.status === 401 && !triedRefresh) {
        triedRefresh = true;
        await refreshToken();
        throw new Error('RETRY'); // force un retry auto de fetchEventSource
      }
      throw new Error(`SSE open failed: ${resp.status}`);
    },
    onmessage: (msg) => {
      // msg.event => 'progress' | 'done' | ...
      // msg.data  => string
      onmessage?.(msg);
      if (msg.event === 'done') {
        controller.abort();
        onclose?.();
      }
    },
    onerror: (err) => {
      // tu peux logger, appliquer un backoff, etc.
      throw err; // laisser le client retenter selon sa stratégie
    },
    // Stratégie de retry (ex: backoff simple)
    retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 15000),
  });

  return () => controller.abort();
}
