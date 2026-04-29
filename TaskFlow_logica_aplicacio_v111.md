# TaskFlow — Lògica de l'aplicació (V110)

> Document generat a partir del fitxer `index_v110.html`.  
> Objectiu: descriure la lògica funcional, l'arquitectura interna i els fluxos principals de TaskFlow perquè sigui més fàcil mantenir, auditar i evolucionar l'aplicació.

---

## 1. Visió general

TaskFlow és una aplicació HTML/JavaScript orientada a gestionar tasques, seguiments, reunions, trucades i relacions amb emails dins un entorn d'Outlook Add-in. La lògica combina:

- Renderitzat dinàmic de targetes de tasques i email.
- Integració amb Outlook/Office.js quan està disponible.
- Preferències d'usuari persistides localment.
- Gestió de dates de seguiment, dates límit, recurrències i restriccions de calendari laboral.
- Internacionalització mitjançant claus `i18n`.
- Utilitats agrupades sota `window.TaskFlow.utils` per reduir contaminació global.

El títol intern detectat és: **`TaskFlow — Panel V110`**.

---

## 2. Arquitectura lògica

L'aplicació està organitzada com una aplicació single-file amb:

1. **HTML base**: estructura principal del panell, capçalera, contenidors de render i overlays.
2. **CSS embegut**: estils de targetes, pop-downs, overlays, vista AVUI, llegenda i integració visual.
3. **JavaScript embegut**: lògica d'estat, renderitzat, persistència, integració Outlook, utilitats i gestió d'accions.
4. **Office.js**: carregat des de Microsoft per operar com a Add-in quan Outlook està disponible.

### Components principals detectats per ID

- `${id}`
- `${t.task_id}`
- `${task.task_id}`
- `${taskId}`
- `bk`
- `btn-menu`
- `call-date`
- `call-flash`
- `call-minutes`
- `call-note`
- `call-people`
- `call-person`
- `call-phone`
- `call-title`
- `cf-category`
- `cf-comment`
- `cf-due`
- `cf-due-weekend-warn`
- `cf-flash`
- `cf-followup-current-date`
- `cf-followup-dom`
- `cf-followup-enabled`
- `cf-followup-finalized`
- `cf-followup-interval-days`
- `cf-followup-interval-weeks`
- `cf-followup-opts`
- `cf-followup-ordinal`
- `cf-followup-ordinal-weekday`
- `cf-followup-section`
- `cf-followup-summary`
- `cf-followup-type`
- `cf-followup-warning`
- `cf-followup-weekday`
- `cf-notes`
- `cf-owner`
- `cf-people`
- `cf-priority`
- `cf-project`
- `cf-projects`
- `cf-provider`
- `cf-providers`
- `cf-repeat-dom`
- `cf-repeat-enabled`
- `cf-repeat-interval-days`
- `cf-repeat-interval-weeks`
- `cf-repeat-opts`
- `cf-repeat-ordinal`
- `cf-repeat-ordinal-weekday`
- `cf-repeat-section`
- `cf-repeat-summary`
- `cf-repeat-type`
- `cf-repeat-weekday`
- `cf-status`
- `cf-time-unit`
- `cf-time-val`
- `cf-title`
- `cnt`
- `db-flash`
- `db-import-input`
- `email-panel`
- … i 63 elements més.

---

## 3. Estat i persistència

### Preferències d'usuari

L'aplicació defineix preferències per defecte, incloent:

- Idioma per defecte.
- Temps per defecte per tipus d'acció: reunió, trucada i tasca.
- Regles d'Outlook per emails marcats, fixats o categoritzats.
- Dies per defecte per seguiment.
- Horari laboral.
- Pas de minuts per reunions.
- Preferències de visualització i comportament amb emails vinculats.

La persistència de preferències utilitza `localStorage`.

### Referències de persistència detectades

- Mètodes `localStorage` detectats: `getItem`, `setItem`.
- Referències IndexedDB/IDB detectades: `IDB`, `INDEXEDDB`, `Idb`, `IndexedDB`, `IndexedDb`, `indexedDB`, `indexeddb`.

---

## 4. Namespace d'utilitats `TaskFlow.utils`

A la versió documentada, les utilitats comunes estan agrupades sota:

```js
window.TaskFlow.utils
```

Això evita deixar helpers comuns com a funcions globals independents i redueix el risc de col·lisions.

### Utilitats principals detectades

- `for`
- `if`

### Rol funcional de les utilitats principals

- `getLocalNowIso()`: genera una data/hora local en format ISO simplificat.
- `addDaysIso(baseIso, days)`: suma dies a una data ISO i retorna una data `YYYY-MM-DD`.
- `getActionTypeFromCategory(category)`: normalitza categories cap a tipus interns com `meeting`, `call`, `followup` o `task`.
- `getLegacyCategoryFromActionType(actionType)`: manté compatibilitat amb categories legacy.
- `normalizeTaskSchema(task)`: normalitza camps d'una tasca, afegeix valors per defecte i manté compatibilitat amb camps antics.
- `migrateTaskCollections()`: aplica normalització a la col·lecció de tasques quan està disponible.

---

## 5. Integració amb Outlook

L'aplicació detecta si s'està executant dins Outlook mitjançant `Office.context.mailbox`.

Quan Outlook està disponible:

1. Es llegeix el context de l'email actual.
2. Es captura informació com assumpte, remitent i identificador del missatge.
3. Es registra un handler `ItemChanged` per reaccionar quan l'usuari canvia d'email en un panell ancorat.
4. Es recarrega el context d'email i es força un `render()` quan cal.

### Patch defensiu de `tfLoadEmailFromOffice`

La versió documentada incorpora un patch final i idempotent sobre `tfLoadEmailFromOffice`:

- Només s'aplica si `window.tfLoadEmailFromOffice` existeix i és una funció.
- Evita aplicar-se dues vegades amb `__taskFlowPrefsPatched`.
- Conserva la funció original a `__taskFlowOriginal`.
- Aplica regles de preferències d'Outlook després de carregar el context.

Aquest enfocament evita el problema del monkey-patch prematur, on el wrapper podia capturar una funció encara no definida.

---

## 6. Model funcional de tasques

La lògica de tasques inclou normalització i compatibilitat amb camps legacy.

### Camps funcionals rellevants

- `action_type`: tipus intern d'acció.
- `expected_minutes`: temps previst.
- `spent_minutes`: temps dedicat.
- `time_minutes`: àlies legacy de temps dedicat.
- `created_at_local`: data/hora local de creació.
- `updated_at_local`: data/hora local d'actualització.
- `followup_date`: data de seguiment.
- `outlook_category`: categoria compatible amb lògica anterior.

### Tipus d'acció principals

- `task`: tasca general.
- `followup`: seguiment.
- `meeting`: reunió.
- `call`: trucada.

La lògica de seguiment afegeix el prefix `SEGUIMENT:` quan correspon i calcula una data de seguiment per defecte si no existeix.

---

## 7. Fluxos principals

### 7.1 Crear tasca sense email

Flux resumit:

1. L'usuari prem el botó de crear tasca sense email.
2. S'obté la configuració del formulari amb mode `create`.
3. Es força `_emailMode = 'none'`.
4. Es netegen valors de títol i notes.
5. Es mostra el pop-down de creació de tasca.

Funcions relacionades:

- `openCreateTaskFormBlank(ev)`
- `toggleTaskCreatePopdown(formConfig, triggerBtn)`
- `mountTaskFormPopdown(formConfig, panel)`
- `closeTaskCreatePopdown()`

### 7.2 Pop-down de creació

La lògica del pop-down:

- Obre/tanca el panell segons l'estat actual.
- Conserva el botó d'origen per retornar focus.
- Tanca amb `Escape`.
- Tanca quan es fa clic fora del panell.
- Fa focus al primer camp del formulari quan s'obre.

Correccions crítiques documentades:

```js
lastCreateButton = triggerBtn || document.activeElement;
```

```js
if (!panel || panel.classList.contains('hidden')) return;
```

### 7.3 Càrrega d'email Outlook

Flux resumit:

1. Outlook exposa l'item actual.
2. TaskFlow llegeix subject, sender i identificador.
3. Es construeix o actualitza `CURRENT_EMAIL`.
4. Es poden aplicar regles de preferències segons si l'email està marcat, fixat o categoritzat.
5. Es refresca la UI amb `render()`.

### 7.4 Regles derivades d'Outlook

Segons preferències:

- Email marcat (`flagged`) pot forçar seguiment o responsable.
- Email fixat (`pinned`) pot forçar seguiment o responsable.
- Email categoritzat pot forçar projecte o comentari.

---

## 8. Vista AVUI

La vista AVUI agrupa informació operativa del dia amb una capçalera compacta. La lògica visual inclou:

- Títol `AVUI`.
- Indicadors numèrics per totals i tipus d'acció.
- Botó de resum AVUI alineat a la dreta.
- Filtres/chips per subconjunts com venciments, seguiments, reunions i trucades.

La vista AVUI està pensada per prioritzar:

- Tasques d'avui.
- Tasques vençudes o anteriors a avui.
- Seguiments previstos.
- Reunions i trucades del dia.

---

## 9. Agrupació i ordenació

La lògica de l'aplicació preveu agrupacions funcionals de tasques. En el comportament recent, quan hi ha un email seleccionat i una tasca associada, aquesta tasca ha de mantenir prioritat visual dins la vista agrupada.

Elements relacionats detectats:

- Tokens funcionals trobats: `TODAY`, `GROUP`, `SUMMARY`, `FOLLOWUP`, `MEETING`, `CALL`, `DEADLINE`, `PROJECT`, `PROVIDER`, `PERSON`, `EMAIL`, `TASK`, `PREFERENCES`, `LEGEND`.
- Classes principals de targetes i agrupadors: `card`, `ghdr`, `gbar`, `gsub`, `legend-body`, `today-view-head`.

---

## 10. Seguiments, recurrències i dates

La lògica de seguiment contempla:

- Seguiment amb data fixa.
- Seguiment periòdic per nombre de dies.
- Càlcul de data derivada.
- Validació de rang entre avui i data límit.
- Ajustos quan una data cau en cap de setmana.
- Missatges d'advertiment i confirmació quan el sistema ajusta dates.

Regles funcionals esperades:

- No permetre seguiment anterior a avui.
- No permetre seguiment posterior a la data límit.
- No activar un nou seguiment si la tasca ja en té un programat.
- Ajustar dates de cap de setmana cap a dia laborable quan correspongui.

---

## 11. Trucades i reunions

TaskFlow tracta `TRUCADA` i `REUNIÓ` com tipus d'acció específics.

### Trucades

La lògica de trucada contempla camps com:

- Data de la trucada.
- Persona a trucar.
- Telèfon.
- Temps previst.

### Reunions

La lògica de reunió contempla com a mínim la classificació com a tipus d'acció i la compatibilitat amb temps previst i data/hora.

Regla funcional important:

- Si una tasca ja és de tipus `TRUCADA` o `REUNIÓ`, el menú `+` de la targeta de tasca no hauria d'oferir tornar a crear la mateixa acció sobre aquella tasca.

---

## 12. Internacionalització i textos

L'aplicació usa un diccionari d'internacionalització amb claus `i18n`. S'han detectat claus de traducció en el fitxer.

### Mostra de claus i18n detectades

- `AT_CALL`
- `AT_FOLLOWUP`
- `AT_MEETING`
- `AT_TASK`
- `CALL_DATE_LBL`
- `CALL_EXPECTED_MINUTES`
- `CALL_PERSON_LBL`
- `CALL_PERSON_PH`
- `CALL_PHONE_LBL`
- `CALL_PHONE_PH`
- `CALL_SAVE`
- `CAT_CALL`
- `CF_BACK`
- `CF_CATEGORY`
- `CF_COMMENT`
- `CF_COMMENT_PH`
- `CF_DUE`
- `CF_FOLLOWUP`
- `CF_NOTES`
- `CF_ORDINAL`
- `CF_PROJECT`
- `CF_PROVIDER`
- `CF_REPEAT_DAYS_Q`
- `CF_REPEAT_LABEL`
- `CF_REPEAT_TYPE`
- `CF_RESPONSIBLE`
- `CF_RESPONSIBLE_HINT`
- `CF_TIME`
- `CF_TITLE`
- `CF_WEEKDAY`
- `CMT`
- `CMT_EMPTY`
- `CMT_NO_ENTRIES`
- `CMT_SAVED`
- `DB_RESET_ERR`
- `DUP_EXISTING`
- `DUP_INTERACTION_PERSON`
- `DUP_WARN_FU`
- `DUP_WARN_MT`
- `EMAIL_ACTIVE`
- `EMAIL_ALREADY_LINKED`
- `EMAIL_LINKED_OK`
- `EML`
- `EMPTY_TITLE`
- `EXIST_FOLLOWUP`
- `EXIST_INFO`
- `FU_ALLOWED_RANGE`
- `FU_ALREADY_EXISTS`
- `FU_DATE_AFTER_DUE`
- `FU_DATE_BEFORE_TODAY`
- `FU_DERIVED_DATE`
- `FU_EMPTY_DATE_OR_DAYS`
- `FU_FIRST_DATE_LBL`
- `FU_OR`
- `FU_PERIODIC_DAYS_LBL`
- `FU_PERIODIC_DAYS_PH`
- `FU_SAVE`
- `FU_TITLE_PH`
- `FU_WEEKEND_ADJUSTED`
- `GL_NO_ASSIGNED`
- `GL_NO_FOLLOWUP`
- `GL_OVERDUE`
- `GL_UPCOMING`
- `GRP_ACTIONTYPE`
- `GRP_BY`
- `GRP_DUEDATE`
- `GRP_RESPONSIBLE`
- `GRP_SHOW_COMPLETED`
- `LEG_DEADLINE`
- `LEG_EMAILS`
- `LEG_PRIORITY`
- `LINK_BACK`
- `LINK_HINT`
- `LINK_SEARCH_PH`
- `MENU_DAILY_SUMMARY`
- `MENU_LANGUAGE`
- `MENU_PREF`
- `MONTHS`
- `MT_PERSON_LBL`
- `MT_PERSON_PH`
- `MT_PLACE_LBL`
- `MT_SAVE`
- `NOTES_ATTACHMENTS`
- `NOTES_DATE`
- `NOTES_FROM`
- `NOTES_SUBJECT`
- `NO_PROJECT`
- `NO_SUBJECT`
- `NO_TASKS_FOUND`
- `OPEN_OUTLOOK`
- `ORD_1`
- `OV_CALL`
- `OV_COMMENTS`
- `OV_CREATE_TASK`
- `OV_EMAILS`
- `PR1`
- `PREFS_TITLE`
- `PREF_LANG`
- `PREF_RESET`
- `PREF_SAVED`
- `PRJ`
- `PRS`
- `PRV`
- `QA_CALL`
- `QA_CALL_SUB`
- `QA_CREATE`
- `QA_FOLLOWUP`
- `QA_LINK`
- `QA_MEETING`
- `REL`
- `REP_INTERVAL_DAYS`
- `REP_MONTHLY_DOM`
- `REP_SUM_DAYS`
- `REP_SUM_DOM`
- `REP_SUM_ORDINAL`
- `REP_SUM_WEEKS`
- `ST_PENDING`
- `TASK_CREATED`
- `TIM`
- `TIM_AMOUNT`
- … i 9 elements més.

Bones pràctiques aplicades o recomanades:

- Evitar literals visibles directes dins funcions quan existeix clau i18n.
- No mostrar claus crues tipus `MENU_XXX` o `LABEL_XXX` a la UI.
- Mantenir cobertura equivalent entre idiomes.
- Després de renderitzats dinàmics, assegurar que els textos visibles provenen de `tr(...)` o de claus `data-i18n`.

---

## 13. Overlays, pop-downs i accessibilitat

Elements funcionals detectats:

- Pop-down de creació de tasca.
- Pop-downs ràpids (`qepop`).
- Overlay inferior (`ov`).
- Backdrop (`bk`).
- Menús secundaris.

Bones pràctiques reflectides en la lògica:

- Ús de `role="dialog"` en el pop-down de creació.
- Ús d'`aria-hidden` per indicar estat ocult/visible.
- Retorn de focus al botó origen en tancar.
- Tancament amb tecla `Escape`.
- Controls perquè pop-downs no surtin dels límits visuals del panell.

---

## 14. Funcions i wrappers detectats

### Funcions globals declarades detectades

- `_renderFollowUpForm`
- `_renderMeetingForm`
- `addDaysIso`
- `addDaysIsoLegacy`
- `addRelation`
- `addTaskSystemComment`
- `adjustToWeekdayIfWeekend`
- `advanceTaskFollowup`
- `applyActiveEmailGroupState`
- `applyI18n`
- `applyPersistenceSnapshot`
- `applyPreferencesI18n`
- `applyQeDueDate`
- `applyQeFollowDate`
- `attachPersistenceHooks`
- `bootTaskFlowApp`
- `buildDailySummary`
- `buildDupConfirmScreen`
- `buildExistingActionsInfo`
- `buildQePopContent`
- `canUseIndexedDb`
- `clearIndexedDbStores`
- `cloneJson`
- `closeOv`
- `closePreferencesPanel`
- `closePrefs`
- `closeQaDrop`
- `closeQePop`
- `closeSecondaryMenu`
- `closeTaskCreatePopdown`
- `collectTaskAuditState`
- `colorFromString`
- `computeFollowupFirstDateFromInputs`
- `computeNextRecurringDate`
- `currentDateTimeLocal`
- `cycleLang`
- `dateStatus`
- `dateToIsoLocal`
- `downloadSnapshotJson`
- `enhanceRenderedTaskCards`
- `ensureCurrentUserInPeople`
- `escHandler`
- `escapeHtml`
- `exportIndexedDbSnapshot`
- `findById`
- `findDuplicateEventsForEmail`
- `findExistingEmailId`
- `findOpenInteractionForPerson`
- `findOrCreateEmail`
- `findOrCreatePerson`
- `findOrCreateProject`
- `findOrCreateProvider`
- `fitQePopInsidePhone`
- `fmtDate`
- `fmtTime`
- `focusByStoredTarget`
- `focusFallback`
- `forceReloadSelectedEmail`
- `getActionTypeFromCategory`
- `getActiveEmail`
- `getActiveEmailSummary`
- `getCurrentOutlookItemId`
- `getCurrentUserDisplayName`
- `getCurrentUserSelfLabel`
- `getDailySummaryDateFromEmail`
- `getDailySummaryTaskDate`
- `getDailySummaryTaskPriorityWeight`
- `getDefaultCallExpectedMinutes`
- `getDueDateSortValue`
- `getEstimatedTimeDisplay`
- `getFocusableElements`
- `getFollowupConfigFromForm`
- `getFollowupDateSortValue`
- `getGroups`
- `getInitials`
- `getLegacyCategoryFromActionType`
- `getLinkedEmailStatusText`
- `getLinkedTasksForTask`
- `getLocalNowIso`
- `getMonthOrdinalDate`
- `getOrdinalNumber`
- `getOutlookItemIdentifierForEmail`
- `getPersistenceSnapshot`
- `getPersistenceStats`
- `getPhoneForPersonName`
- `getPrefsEls`
- `getPrimaryLinkedTaskForActiveEmail`
- `getPrimaryProjectIdForActiveEmail`
- `getRelated`
- `getRelatedIds`
- `getRepeatConfig`
- `getRepeatSummaryFromConfig`
- `getRestTaskDateRank`
- `getSelectedEmailContextTaskIds`
- `getSuggestedTasksForActiveEmailThread`
- `getTaskComments`
- `getTaskDateForSort`
- `getTaskDueDateForFollowup`
- `getTaskEmails`
- `getTaskFollowup`
- `getTaskFollowupConflict`
- `getTaskFormConfig`
- `getTaskFormPayload`
- `getTaskIdsLinkedToEmailId`
- `getTaskPeople`
- `getTaskProject`
- `getTaskProviders`
- `getTasksLinkedToActiveEmail`
- `getTasksLinkedToEmailId`
- `getTodayGroupActionRank`
- `getTodayGroupPriorityWeight`
- `getTodayGroupTaskDate`
- `getUserCommentCount`
- `getWeekdayIndex`
- `handleDueDateChange`
- `handleGlobalKeydown`
- `handleIndexedDbImport`
- `hasPersistenceData`
- `hasSystemComments`
- `isFocusable`
- … i 144 elements més.

### Funcions assignades a `window` detectades

- `closePreferencesPanel`
- `fitQePopInsidePhone`
- `openPreferencesPanel`
- `openTimeRegister`
- `tfLoadEmailFromOffice`

---

## 15. Riscos tècnics i punts de manteniment

### 15.1 Single-file molt gran

El fitxer concentra HTML, CSS, JS, icones i dades embegudes. Això facilita proves ràpides però dificulta:

- Refactorització.
- Revisions de regressió.
- Test unitari.
- Control granular de canvis.

### 15.2 Monkey-patches

Els wrappers sobre funcions existents són útils per no tocar el core, però convé que siguin:

- Idempotents.
- Aplicats després de la definició real.
- Documentats amb versió i finalitat.
- Reversibles o traçables amb referència a la funció original.

### 15.3 Helpers globals

La migració a `window.TaskFlow.utils` redueix el risc de col·lisions, però encara convé continuar encapsulant nous helpers dins namespaces equivalents.

### 15.4 Render dinàmic

Qualsevol patch que observi o forci renderitzats ha d'evitar interferir amb `renderCardEmail`, `AGRUPAR PER` o altres zones dinàmiques. És preferible corregir causes concretes abans que aplicar observadors globals agressius.

---

## 16. Checklist de regressió recomanat

Després de qualsevol canvi funcional, validar:

- [ ] Obre correctament el renderCard Email.
- [ ] Funciona la vista `AGRUPAR PER`.
- [ ] El menú `+` del renderCard Tasca obre i tanca correctament.
- [ ] El menú `+` del renderCard Email conserva la lògica esperada.
- [ ] Crear tasca sense email obre el pop-down correcte.
- [ ] `Escape` tanca pop-downs i retorna focus.
- [ ] La vista AVUI mostra només les tasques esperades.
- [ ] La tasca associada a l'email seleccionat apareix prioritzada.
- [ ] Les dates de seguiment no poden ser anteriors a avui.
- [ ] Les dates de seguiment no superen la data límit.
- [ ] Les dates en cap de setmana generen advertiment/acceptació.
- [ ] Les traduccions no mostren claus crues.
- [ ] Outlook `ItemChanged` refresca el context d'email.
- [ ] `tfLoadEmailFromOffice` conserva el patch defensiu i no es duplica.

---

## 17. Convencions de manteniment

- Comentaris de codi en castellà.
- Explicacions funcionals i documentació en català quan sigui per ús intern de Xavier.
- Evitar `ñ`, accents i caràcters especials en noms de variables.
- Evitar helpers globals nous si poden anar a `window.TaskFlow.utils` o a un namespace específic.
- Mantenir versions incrementals i títol intern coherent.
- Incloure checklist de regressió quan hi hagi canvis sobre render, Outlook o menús `+`.

---

## 18. Resum executiu

La lògica de TaskFlow està centrada en convertir emails i context operatiu en tasques accionables, amb suport per seguiments, trucades, reunions, venciments, agrupació visual i preferències d'usuari. La versió documentada reforça dos aspectes importants de mantenibilitat:

1. **Patch defensiu de `tfLoadEmailFromOffice`**, per evitar regressions per ordre de càrrega.
2. **Namespace `window.TaskFlow.utils`**, per encapsular helpers comuns i reduir contaminació global.

Aquest document pot servir com a base per futures revisions, refactors o separació progressiva del single-file en mòduls.
