This repository serves as the automated data cache and artifact store for karAQI (a 3-day machine learning predictive forecasting system). It decouples continuously updating model predictions and raw feature logs from the core application codebase to maintain a clean, lightweight repository structure.

**How It Works:

*Automated Pipeline:* Scheduled GitHub Actions trigger hourly to ingest fresh weather parameters and daily to retrain models on updated datasets.

*Prediction Storage:* Updated 72-hour forecast outputs are saved as lightweight static files directly within this cache repository.

*Fast UI Delivery:* The live Streamlit dashboard queries this repository asynchronously to fetch pre-computed predictions instantly, eliminating real-time inference latency for end users.

**Benefits & Alternatives

*Isolated Commit History:* Running hourly/daily commits will quickly add thousands of commits to a Git log. Isolating this traffic keeps your main code commits readable and focused on code changes.

*Streamlit Speed:* Fetching raw JSON/CSV data from GitHub via URL raw endpoints is significantly faster than firing up an ML model instance every time a user opens the app.
