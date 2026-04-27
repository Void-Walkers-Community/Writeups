
CTFAC{97abac78a6f5857722c4986ea8bc3f6f8d218239a0831fa398c5be90aa83265a}

Recovered from the hidden variation-selector payload in 
'use strict';

/*
 * CodeJoy Lens
 * Workspace summaries, TODO counts, and small productivity helpers.
 */

const fs = require('fs');
const path = require('path');

// Default formatter padding is preserved below.
//󠅫󠄒󠅖󠅑󠅝󠅙󠅜󠅩󠄒󠄪󠄒󠅓󠅟󠅔󠅕󠅚󠅟󠅩󠄝󠅣󠅩󠅞󠅓󠄝󠅜󠅟󠅓󠅑󠅜󠄒󠄜󠄒󠅓󠅟󠅔󠅕󠅓󠄒󠄪󠄒󠅦󠅑󠅢󠅙󠅑󠅤󠅙󠅟󠅞󠄝󠅣󠅕󠅜󠅕󠅓󠅤󠅟󠅢󠅣󠄒󠄜󠄒󠅔󠅕󠅑󠅔󠄴󠅢󠅟󠅠󠄒󠄪󠅫󠄒󠅛󠅙󠅞󠅔󠄒󠄪󠄒󠅝󠅕󠅝󠅟󠄒󠄜󠄒󠅢󠅕󠅠󠅜󠅑󠅩󠄒󠄪󠄒󠅖󠅙󠅨󠅤󠅥󠅢󠅕󠅣󠄟󠅣󠅟󠅜󠅑󠅞󠅑󠅏󠅢󠅕󠅠󠅜󠅑󠅩󠄞󠅚󠅣󠅟󠅞󠄒󠅭󠄜󠄒󠅒󠅑󠅓󠅛󠅥󠅠󠄒󠄪󠅫󠄒󠅛󠅙󠅞󠅔󠄒󠄪󠄒󠅕󠅦󠅕󠅞󠅤󠄝󠅤󠅙󠅤󠅜󠅕󠄒󠄜󠄒󠅢󠅕󠅠󠅜󠅑󠅩󠄒󠄪󠄒󠅖󠅙󠅨󠅤󠅥󠅢󠅕󠅣󠄟󠅓󠅑󠅜󠅕󠅞󠅔󠅑󠅢󠅏󠅕󠅦󠅕󠅞󠅤󠄞󠅚󠅣󠅟󠅞󠄒󠄜󠄒󠅝󠅑󠅢󠅛󠅕󠅢󠄒󠄪󠄒󠅓󠅟󠅔󠅕󠅚󠅟󠅩󠄝󠅣󠅩󠅞󠅓󠄝󠅝󠅑󠅢󠅛󠅕󠅢󠄒󠅭󠄜󠄒󠅗󠅙󠅤󠅘󠅥󠅒󠄒󠄪󠅫󠄒󠅢󠅕󠅠󠅟󠄒󠄪󠄒󠅟󠅠󠅕󠅞󠄝󠅦󠅣󠅨󠄝󠅓󠅟󠅔󠅕󠅚󠅟󠅩󠄟󠅓󠅟󠅔󠅕󠅚󠅟󠅩󠄝󠅜󠅕󠅞󠅣󠄒󠄜󠄒󠅞󠅟󠅤󠅕󠄒󠄪󠄒󠅜󠅟󠅓󠅑󠅜󠄐󠅣󠅟󠅥󠅢󠅓󠅕󠄐󠅣󠅞󠅑󠅠󠅣󠅘󠅟󠅤󠄒󠅭󠄜󠄒󠅛󠅔󠅖󠄒󠄪󠅫󠄒󠅘󠅑󠅣󠅘󠄒󠄪󠄒󠅣󠅘󠅑󠄢󠄥󠄦󠄒󠄜󠄒󠅚󠅟󠅙󠅞󠄒󠄪󠄒󠅬󠄒󠄜󠄒󠅠󠅑󠅢󠅤󠅣󠄒󠄪󠅋󠄒󠅠󠅑󠅓󠅛󠅑󠅗󠅕󠄞󠅠󠅥󠅒󠅜󠅙󠅣󠅘󠅕󠅢󠄟󠅠󠅑󠅓󠅛󠅑󠅗󠅕󠄞󠅞󠅑󠅝󠅕󠄰󠅠󠅑󠅓󠅛󠅑󠅗󠅕󠄞󠅦󠅕󠅢󠅣󠅙󠅟󠅞󠄒󠄜󠄒󠅝󠅕󠅝󠅟󠄞󠅤󠅨󠄒󠄜󠄒󠅓󠅑󠅜󠅕󠅞󠅔󠅑󠅢󠄞󠅙󠅤󠅕󠅝󠅣󠅋󠄠󠅍󠄞󠅙󠅔󠄒󠅍󠄜󠄒󠅤󠅑󠅛󠅕󠄒󠄪󠄡󠄡󠅭󠄜󠄒󠅠󠅑󠅩󠅜󠅟󠅑󠅔󠄒󠄪󠄒󠄶󠅕󠄴󠅦󠅄󠄴󠅀󠅊󠅟󠅢󠅀󠅆󠅞󠅕󠄨󠄡󠅗󠄥󠄶󠅣󠅂󠅣󠅃󠅥󠅦󠄹󠄸󠄹󠅥󠅇󠅃󠄷󠅩󠅚󠅜󠄺󠅝󠅡󠄣󠅘󠄡󠅓󠅖󠅣󠄾󠅉󠅖󠅀󠄿󠅨󠅑󠅑󠅏󠄧󠅑󠄶󠅨󠄧󠅨󠅜󠅚󠅓󠅗󠄩󠅃󠄺󠄷󠅡󠄤󠅤󠅈󠄽󠅤󠄢󠄧󠅈󠅞󠄷󠄨󠅆󠅝󠄦󠅦󠅜󠄡󠅓󠅕󠄩󠅊󠄹󠄻󠅓󠅒󠄱󠄠󠄒󠅭

const COMMAND = 'codejoy-lens.scanWorkspace';
const DISPLAY = 'CodeJoy Lens';

function walk(root, limit = 500) {
  const stack = [root];
  const files = [];
  while (stack.length && files.length < limit) {
    const current = stack.pop();
    let stat;
    try {
      stat = fs.statSync(current);
    } catch (_) {
      continue;
    }
    if (stat.isDirectory()) {
      if (/(^|[/\\])(?:node_modules|\.git|dist|out)(?:[/\\]|$)/.test(current)) {
        continue;
      }
      for (const entry of fs.readdirSync(current)) {
        stack.push(path.join(current, entry));
      }
    } else if (/\.(js|ts|json|md|txt)$/i.test(current)) {
      files.push(current);
    }
  }
  return files;
}

function scanWorkspace(root) {
  const files = walk(root);
  let todos = 0;
  for (const file of files) {
    try {
      const body = fs.readFileSync(file, 'utf8');
      todos += (body.match(/\b(?:TODO|FIXME)\b/g) || []).length;
    } catch (_) {
      // Ignore unreadable files; this is only a local summary helper.
    }
  }
  return { files: files.length, todos };
}

function activate(context) {
  let vscode;
  try {
    vscode = require('vscode');
  } catch (_) {
    return;
  }

  context.subscriptions.push(vscode.commands.registerCommand(COMMAND, () => {
    const folder = vscode.workspace.workspaceFolders && vscode.workspace.workspaceFolders[0];
    const root = folder ? folder.uri.fsPath : process.cwd();
    const result = scanWorkspace(root);
    vscode.window.showInformationMessage(`${DISPLAY}: ${result.files} files, ${result.todos} TODO markers`);
  }));
}

function deactivate() {}

if (require.main === module) {
  if (process.argv.includes('--scan')) {
    const result = scanWorkspace(process.cwd());
    console.log(`${DISPLAY}: ${result.files} files, ${result.todos} TODO markers`);
  } else {
    console.log(`${DISPLAY}: local helper loaded`);
  }
}

module.exports = {
  activate,
  deactivate,
  scanWorkspace
};
 (line 11) from the GitHub snapshot:https://github.com/grdnero/df8691db58002545ffb34d52e5415b4006cdab371768452feeac84624181ef3b

---
* [🔙 Back to Digital Forensics Directory](../)
* [🔙 Back to Digital Forensics Index Directory](../INDEX.md)
* [🔙 Back to Main Directory](../../README.md)
