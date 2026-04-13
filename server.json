const express  = require('express');
const fetch    = require('node-fetch');
const app      = express();

app.use(express.urlencoded({ extended: true }));

// Allow all CORS
app.use(function(req, res, next) {
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Methods', 'POST, GET, OPTIONS');
    res.header('Access-Control-Allow-Headers', 'Content-Type');
    if (req.method === 'OPTIONS') {
        return res.sendStatus(200);
    }
    next();
});

// Replace these with your actual Firebase credentials
const PROJECT_ID = 'cmm702-clicklogs';
const API_KEY    = 'AIzaSyCIN9fvOWEvsR2HZZqhf7evdNf1JST5uqc';

const FIRESTORE  = 'https://firestore.googleapis.com/v1/projects/'
                 + PROJECT_ID
                 + '/databases/(default)/documents/tap_logs?key='
                 + API_KEY;

// POST endpoint — receives tap data from index.html
app.post('/saveTaps.php', async function(req, res) {

    const sessionId = req.body.id  || null;
    const platform  = req.body.var || null;
    const tapsRaw   = req.body.taps || '[]';

    if (!sessionId || !platform) {
        return res.status(400).send('Bad Request: Missing session id or platform');
    }

    let taps = [];
    try {
        taps = JSON.parse(tapsRaw);
    } catch(e) {
        return res.status(400).send('Bad Request: Invalid taps data');
    }

    const serverTime = new Date().toISOString();
    let savedCount   = 0;

    for (const tap of taps) {

        const duration = tap.endTimestamp - tap.startTimestamp;
        if (duration <= 0) continue;

        const document = {
            fields: {
                session_id:        { stringValue:  sessionId.toString() },
                platform:          { stringValue:  platform.toLowerCase() },
                tapSequenceNumber: { integerValue: tap.tapSequenceNumber.toString() },
                startTimestamp:    { integerValue: tap.startTimestamp.toString() },
                endTimestamp:      { integerValue: tap.endTimestamp.toString() },
                duration_ms:       { integerValue: duration.toString() },
                interface:         { stringValue:  tap.interface },
                interfaceSequence: { integerValue: tap.interfaceSequence.toString() },
                createdAt:         { stringValue:  serverTime }
            }
        };

        try {
            const response = await fetch(FIRESTORE, {
                method:  'POST',
                headers: { 'Content-Type': 'application/json' },
                body:    JSON.stringify(document)
            });
            if (response.ok) savedCount++;
        } catch(e) {
            console.log('Firestore error:', e.message);
        }
    }

    if (savedCount > 0) {
        res.send('Data saved successfully');
    } else {
        res.status(500).send('Error: No taps saved');
    }
});

// GET endpoint — for browser testing
app.get('/saveTaps.php', function(req, res) {
    res.send('Bad Request: Missing session id or platform');
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, function() {
    console.log('Server running on port ' + PORT);
});