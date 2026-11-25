# 2-minute demo script

Before you start: Docker is running, Ollama has `llama3.2`, both grading workflows and the error handler are published, your email inbox is open in one tab, and the `class_record` data table is open in another.

**0:00 - The problem (15 seconds)**
"Grading essays against a rubric is slow, and late penalties are done by hand. A small AI can draft a grade, but it can be wrong. So in this project the AI only drafts, and the teacher decides. The AI runs on my computer with Ollama, so no student work goes to a cloud."

**0:15 - Show Workflow 1 (20 seconds)**
Open "AI Grading and Class Record Automation". "A hand-in comes in through a webhook. Ollama scores each rubric criterion. Then a Code node checks the AI. It works out the total itself, limits every score to the maximum, and sends the draft to manual grading if the answer tries to trick the AI, or if the AI contradicts itself."

**0:35 - Send a hand-in (25 seconds)**
Run `curl -X POST http://localhost:5678/webhook/gleams-submitted -H "Content-Type: application/json" -d @docs/sample-hand-in.json`. Show the answer `{"status":"received"...}`. "The webhook answers at once, and the grading runs after." Open the execution when it finishes.

**1:00 - The teacher email (15 seconds)**
Show the email: rubric table, draft total, Approve and Reject links. "This is only a draft. Nothing is in the class record yet."

**1:15 - Confirm and approve (25 seconds)**
Click Approve. Show the confirm page. "Opening the link only shows the draft. This stops email scanners from approving by accident." Press **Confirm**. Show the page: AI total, late penalty, final score. Open `class_record` and show the new row. "The late penalty and the final score are worked out by code, not by the AI."

**1:40 - Safe behavior (15 seconds)**
Press Back and click Confirm again, or open the same link. Show "Already handled". "A second click changes nothing." Then show the execution of the prompt-injection test (`fake-sub-005`) in `grade_drafts` with `status = manual_required`.

**1:55 - Close (5 seconds)**
"Local AI, a safety check on the AI, teacher approval, and an error handler in one n8n project."

If someone asks: the late penalty is 10% of the maximum per day, and it is a constant in the Evaluate Decision node. The teacher cannot edit scores in the page yet. The small model can give a wrong score, and that is why the approval step exists.
