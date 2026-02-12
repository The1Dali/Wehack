# EduFlow

> AI-powered intelligent timetable generation system that transforms university scheduling from chaos to clarity

## The Problem

**86% of college students struggle with time management. 54% experience increased stress because of it.**

But here's what nobody talks about: it's not entirely their fault. The system is broken from day one.

Current university scheduling creates:
- Wasted time: Students lose 10+ hours per week in unproductive schedule gaps
- Unnecessary stress: 54% of students stressed due to poor time management
- Administrative burden: 40+ hours of manual scheduling work per semester
- Faculty frustration: Weeks of back-and-forth emails about availability

**EduFlow solves this with intelligent, AI-powered timetable generation.**

---

## The Solution

An agentic AI system built with LangGraph that generates optimized schedules by considering:
- Professor availability and preferences
- Student scheduling preferences
- Room availability and capacity constraints
- Course prerequisites and requirements
- Department guidelines and policies

### Key Features

**For Students:**
- Optimized schedules with minimal gaps (average 45min vs. 3 hours)
- Personalized to your preferences (no 8am classes, consolidated days, etc.)
- Transparent explanations for why you got each time slot
- Easy section swap requests

**For Professors:**
- 2-minute availability input (vs. weeks of emails)
- Schedules that respect research and personal time
- Natural language input: "I need Wednesdays free for research"
- Visibility into scheduling decisions

**For Administrators:**
- 90% reduction in scheduling time (40 hours to 4 hours)
- Smart conflict detection and resolution suggestions
- Data-driven insights on course demand and trends
- One-click schedule approval with instant impact analysis

---

## Tech Stack

### Frontend
- **Framework:** Flutter
- **State Management:** Riverpod
- **UI Components:** Material Design 3
- **Platforms:** Web, iOS, Android, Desktop

### Backend
- **Database:** PostgreSQL
- **API:** RESTful API (FastAPI/Python)
- **Authentication:** JWT-based auth
- **ORM:** SQLAlchemy

### AI Agent
- **Framework:** LangGraph
- **LLM Provider:** Google Gemini (via LangChain)
- **Agent Type:** Multi-agent system with planning and execution
- **Tools:** Custom constraint satisfaction tools
- **Memory:** PostgreSQL-backed conversation history

### DevOps
- **Containerization:** Docker & Docker Compose
- **CI/CD:** GitHub Actions
- **Hosting:** TBD (Railway, Render, or cloud provider)

---

## Project Structure
```
unitopia-scheduler/
├── frontend/                 # Flutter application
│   ├── lib/
│   │   ├── models/          # Data models
│   │   ├── services/        # API services
│   │   ├── providers/       # State management (Riverpod)
│   │   ├── screens/         # UI screens
│   │   │   ├── student/     # Student dashboard & views
│   │   │   ├── professor/   # Professor availability input
│   │   │   └── admin/       # Admin review dashboard
│   │   ├── widgets/         # Reusable components
│   │   └── utils/           # Helper functions
│   ├── assets/              # Images, fonts, etc.
│   └── pubspec.yaml
│
├── backend/                 # FastAPI server
│   ├── app/
│   │   ├── api/             # API endpoints
│   │   │   └── routes/      # Route handlers
│   │   ├── models/          # SQLAlchemy models
│   │   ├── schemas/         # Pydantic schemas
│   │   ├── services/        # Business logic
│   │   ├── core/            # Config, security
│   │   └── db/              # Database connection
│   ├── alembic/             # Database migrations
│   ├── requirements.txt
│   └── main.py
│
├── ai-agent/                # LangGraph scheduling agent
│   ├── agents/
│   │   ├── scheduler_graph.py      # Main LangGraph workflow
│   │   ├── planner_agent.py        # Planning agent
│   │   ├── optimizer_agent.py      # Optimization agent
│   │   ├── validator_agent.py      # Validation agent
│   │   └── explainer_agent.py      # Explanation agent
│   ├── tools/
│   │   ├── constraint_checker.py   # Check scheduling constraints
│   │   ├── conflict_resolver.py    # Resolve scheduling conflicts
│   │   ├── database_query.py       # Query PostgreSQL
│   │   └── optimization.py         # Constraint satisfaction algorithms
│   ├── prompts/             # LLM prompt templates
│   ├── state/               # LangGraph state definitions
│   ├── checkpoints/         # Agent checkpoint storage
│   └── requirements.txt
│
├── database/
│   ├── schema.sql           # Database schema
│   └── seed.sql             # Sample data
│
├── docs/
│   ├── API.md               # API documentation
│   ├── ARCHITECTURE.md      # System architecture
│   ├── LANGGRAPH.md         # LangGraph agent design
│   └── DEPLOYMENT.md        # Deployment guide
│
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## LangGraph Agent Architecture

### Multi-Agent System Design

The scheduling system uses LangGraph to orchestrate multiple specialized agents:

**1. Planner Agent**
- Analyzes incoming scheduling request
- Breaks down into sub-tasks
- Identifies constraints and priorities
- Creates execution plan

**2. Optimizer Agent**
- Executes constraint satisfaction algorithms
- Balances competing preferences
- Generates candidate schedules
- Scores solutions based on optimization criteria

**3. Validator Agent**
- Checks for hard constraint violations
- Identifies soft constraint trade-offs
- Flags conflicts for review
- Ensures regulatory compliance

**4. Explainer Agent**
- Generates human-readable explanations
- Answers "why" questions about scheduling decisions
- Provides transparency into AI reasoning
- Creates admin review summaries

### LangGraph Workflow
```python
# Simplified graph structure
from langgraph.graph import StateGraph, END

workflow = StateGraph(SchedulingState)

# Add nodes
workflow.add_node("planner", planner_agent)
workflow.add_node("optimizer", optimizer_agent)
workflow.add_node("validator", validator_agent)
workflow.add_node("explainer", explainer_agent)

# Define edges
workflow.add_edge("planner", "optimizer")
workflow.add_conditional_edges(
    "optimizer",
    should_validate,
    {
        "validate": "validator",
        "regenerate": "optimizer",
    }
)
workflow.add_edge("validator", "explainer")
workflow.add_edge("explainer", END)

# Set entry point
workflow.set_entry_point("planner")

# Compile
app = workflow.compile(checkpointer=checkpointer)
```

### Agent State Management
```python
from typing import TypedDict, List, Dict, Annotated
from langgraph.graph import add_messages

class SchedulingState(TypedDict):
    messages: Annotated[List, add_messages]
    semester_id: str
    constraints: Dict
    professor_availability: List[Dict]
    student_preferences: List[Dict]
    room_data: List[Dict]
    candidate_schedules: List[Dict]
    conflicts: List[Dict]
    selected_schedule: Dict
    explanations: Dict
    validation_status: str
```

### Custom Tools

**Constraint Checker Tool**
```python
from langchain.tools import tool

@tool
def check_constraints(schedule: dict, constraints: dict) -> dict:
    """
    Validates a proposed schedule against hard and soft constraints.
    Returns violations and severity levels.
    """
    violations = []
    
    # Check hard constraints
    if has_double_booking(schedule):
        violations.append({
            "type": "hard",
            "constraint": "no_double_booking",
            "severity": "critical"
        })
    
    # Check soft constraints
    if has_excessive_gaps(schedule):
        violations.append({
            "type": "soft",
            "constraint": "minimize_gaps",
            "severity": "medium"
        })
    
    return {"violations": violations, "valid": len([v for v in violations if v["type"] == "hard"]) == 0}
```

**Database Query Tool**
```python
@tool
def query_professor_availability(professor_id: str, semester_id: str) -> dict:
    """
    Retrieves professor availability from PostgreSQL database.
    """
    query = """
        SELECT day_of_week, start_time, end_time, preference_level
        FROM professor_availability
        WHERE professor_id = %s AND semester_id = %s
    """
    results = db.execute(query, (professor_id, semester_id))
    return {"availability": results}
```

### Example Agent Interaction
```python
# Planner Agent receives request
planner_prompt = """
You are a scheduling planner. Analyze this request and create an execution plan.

Request: Generate schedule for CS Department, Spring 2026
Constraints:
- 15 professors with varying availability
- 45 courses (30 lectures, 15 labs)
- Student preferences: 60% want no 8am classes
- 3 specialized labs with limited equipment

Create a step-by-step plan to generate an optimal schedule.
"""

# Optimizer Agent generates solutions
optimizer_prompt = """
You are a scheduling optimizer. Generate 3 candidate schedules that:
1. Respect all professor availability (hard constraint)
2. Minimize average student gap time (soft constraint)
3. Avoid 8am classes where possible (soft constraint)
4. Ensure lab equipment availability (hard constraint)

Use the constraint_checker tool to validate each candidate.
"""

# Explainer Agent creates transparency
explainer_prompt = """
You are a scheduling explainer. For the selected schedule, explain:
- Why Prof. Smith teaches at 9am MWF
- Why CHEM 101 lab is Tuesday 2-5pm
- What trade-offs were made
- Which preferences were prioritized

Make explanations clear for non-technical users.
"""
```

### Memory and Checkpointing

LangGraph uses PostgreSQL-backed checkpointing to:
- Save agent state at each step
- Enable resume after interruptions
- Support human-in-the-loop approvals
- Track decision history for auditing
```python
from langgraph.checkpoint.postgres import PostgresSaver

# Configure checkpointer
checkpointer = PostgresSaver(
    connection_string=os.getenv("DATABASE_URL")
)

# Compile graph with checkpointing
app = workflow.compile(checkpointer=checkpointer)

# Invoke with thread_id for persistence
result = app.invoke(
    {"messages": [initial_message]},
    config={"configurable": {"thread_id": "semester_spring2026"}}
)
```

---

## Database Schema

### Core Tables

**users**
- `id` (UUID, Primary Key)
- `email` (VARCHAR, Unique)
- `password_hash` (VARCHAR)
- `role` (ENUM: 'student', 'professor', 'admin')
- `first_name`, `last_name` (VARCHAR)
- `created_at`, `updated_at` (TIMESTAMP)

**professors**
- `id` (UUID, Primary Key)
- `user_id` (UUID, Foreign Key to users)
- `department_id` (UUID, Foreign Key to departments)
- `office_location` (VARCHAR)

**professor_availability**
- `id` (UUID, Primary Key)
- `professor_id` (UUID, Foreign Key to professors)
- `semester_id` (UUID, Foreign Key to semesters)
- `day_of_week` (INTEGER: 0-6)
- `start_time`, `end_time` (TIME)
- `preference_level` (ENUM: 'required', 'preferred', 'available')

**courses**
- `id` (UUID, Primary Key)
- `code` (VARCHAR: e.g., "CS101")
- `name` (VARCHAR)
- `department_id` (UUID, Foreign Key)
- `credits` (INTEGER)
- `requires_lab` (BOOLEAN)

**sections**
- `id` (UUID, Primary Key)
- `course_id` (UUID, Foreign Key to courses)
- `professor_id` (UUID, Foreign Key to professors)
- `semester_id` (UUID, Foreign Key to semesters)
- `section_number` (VARCHAR: e.g., "01")
- `capacity` (INTEGER)
- `enrolled_count` (INTEGER)

**schedule_slots**
- `id` (UUID, Primary Key)
- `section_id` (UUID, Foreign Key to sections)
- `room_id` (UUID, Foreign Key to rooms)
- `day_of_week` (INTEGER)
- `start_time`, `end_time` (TIME)
- `generated_by_ai` (BOOLEAN)
- `ai_confidence_score` (DECIMAL)

**student_preferences**
- `id` (UUID, Primary Key)
- `student_id` (UUID, Foreign Key to students)
- `semester_id` (UUID, Foreign Key to semesters)
- `no_early_classes` (BOOLEAN)
- `early_class_threshold` (TIME)
- `minimize_gaps` (BOOLEAN)
- `consolidate_days` (BOOLEAN)
- `preferred_days` (JSON)

**langgraph_checkpoints**
- `id` (UUID, Primary Key)
- `thread_id` (VARCHAR)
- `checkpoint_id` (VARCHAR)
- `state` (JSON)
- `metadata` (JSON)
- `created_at` (TIMESTAMP)

**agent_execution_logs**
- `id` (UUID, Primary Key)
- `thread_id` (VARCHAR)
- `agent_name` (VARCHAR)
- `tool_calls` (JSON)
- `input_state` (JSON)
- `output_state` (JSON)
- `execution_time_ms` (INTEGER)
- `created_at` (TIMESTAMP)

---

## Getting Started

### Prerequisites

- **Flutter SDK** (>= 3.0.0)
- **Python** (>= 3.11)
- **PostgreSQL** (>= 14)
- **Docker** (optional, recommended)
- **Google Gemini API Key** (for LangGraph LLM)

### Installation

1. **Clone the repository**
```bash
   git clone https://github.com/yourusername/unitopia-scheduler.git
   cd unitopia-scheduler
```

2. **Set up environment variables**
```bash
   cp .env.example .env
   # Edit .env and add your credentials:
   # DATABASE_URL=postgresql://user:password@localhost:5432/unitopia
   # GOOGLE_API_KEY=your_gemini_api_key
   # JWT_SECRET=your_jwt_secret
```

3. **Start the database**
```bash
   docker-compose up -d postgres
```

4. **Set up Python virtual environment and install dependencies**
```bash
   # Backend
   cd backend
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   
   # AI Agent
   cd ../ai-agent
   pip install -r requirements.txt
```

5. **Run database migrations**
```bash
   cd backend
   alembic upgrade head
```

6. **Seed sample data (optional)**
```bash
   python scripts/seed_database.py
```

7. **Start the backend**
```bash
   cd backend
   uvicorn app.main:app --reload
   # Backend runs on http://localhost:8000
```

8. **Start the Flutter app**
```bash
   cd frontend
   flutter pub get
   flutter run -d chrome  # For web
   # or: flutter run -d macos  # For desktop
```

---

## API Endpoints

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login user
- `POST /api/auth/refresh` - Refresh JWT token

### Professor Endpoints
- `POST /api/professors/availability` - Submit availability
- `GET /api/professors/{id}/availability` - Get availability
- `PUT /api/professors/availability/{id}` - Update availability

### Scheduling Endpoints
- `POST /api/schedules/generate` - Trigger AI schedule generation
- `GET /api/schedules/{semester_id}` - Get generated schedule
- `POST /api/schedules/{id}/approve` - Admin approve schedule
- `GET /api/schedules/{id}/conflicts` - Get scheduling conflicts
- `GET /api/schedules/{id}/explanations` - Get AI explanations

### Student Endpoints
- `POST /api/students/preferences` - Submit scheduling preferences
- `GET /api/students/{id}/schedule` - Get student schedule
- `POST /api/students/swap-request` - Request section swap

### Admin Endpoints
- `GET /api/admin/dashboard` - Get admin dashboard data
- `GET /api/admin/analytics` - Get scheduling analytics
- `POST /api/admin/override` - Manual schedule override

---

## Development Workflow

### Running Tests
```bash
# Backend tests
cd backend
pytest

# Agent tests
cd ai-agent
pytest tests/

# Frontend tests
cd frontend
flutter test
```

### Running LangGraph Agent Locally
```bash
cd ai-agent
python -m agents.scheduler_graph --semester-id "spring2026"
```

### LangSmith Tracing (Optional)

Enable LangSmith for agent debugging:
```bash
export LANGCHAIN_TRACING_V2=true
export LANGCHAIN_API_KEY=your_langsmith_api_key
export LANGCHAIN_PROJECT=unitopia-scheduler
```

---

## Deployment

### Docker Deployment
```bash
# Build all services
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f
```

### Environment Variables

**Required:**
- `DATABASE_URL` - PostgreSQL connection string
- `GOOGLE_API_KEY` - Gemini API key for LangGraph
- `JWT_SECRET` - Secret for JWT token generation

**Optional:**
- `LANGCHAIN_TRACING_V2` - Enable LangSmith tracing
- `LANGCHAIN_API_KEY` - LangSmith API key
- `REDIS_URL` - Redis for caching (future)

---

## Roadmap

### Phase 1: Core Scheduling (Hackathon MVP)
- Professor availability input interface
- LangGraph multi-agent scheduling system
- Student schedule viewing dashboard
- Admin review and approval interface
- Basic constraint satisfaction

### Phase 2: Advanced Optimization
- Multi-objective optimization (gaps, walking distance, preferences)
- Room allocation integration
- Section swap marketplace
- Real-time conflict detection
- Enhanced LangGraph tools

### Phase 3: Exam Scheduling
- Exam timetable generation agent
- No student has 3+ exams in one day constraint
- Proper spacing between exams
- Integration with class schedule

### Phase 4: Campus Ecosystem
- Campus events calendar integration
- Study time blocking recommendations
- Club meeting scheduler
- Unified student timeline view

### Phase 5: Intelligence and Analytics
- Predictive modeling for course demand
- Semester-over-semester learning
- Department-wide analytics dashboard
- Student success correlation analysis
- Agent performance optimization

---

## Architecture Decisions

### Why LangGraph?

**Advantages over direct LLM API calls:**

1. **Structured Workflow:** LangGraph provides a graph-based execution model, making complex multi-step scheduling logic explicit and debuggable

2. **Agent Orchestration:** Multiple specialized agents (planner, optimizer, validator, explainer) can work together with defined handoffs

3. **Checkpointing:** Built-in state persistence allows resuming from interruptions and human-in-the-loop approvals

4. **Tool Integration:** Easy integration of custom constraint satisfaction algorithms and database queries

5. **Observability:** LangSmith integration provides detailed tracing of agent decision-making

6. **Scalability:** Graph-based design scales better than monolithic prompt chains as complexity grows

### Why PostgreSQL?

- Relational data model fits scheduling constraints naturally
- ACID compliance ensures data consistency
- Native JSON support for flexible agent state storage
- LangGraph checkpoint storage compatibility
- Proven performance at university scale

### Why Flutter?

- Single codebase for web, mobile, and desktop
- Rich UI components out of the box
- Strong typing with Dart
- Hot reload for rapid development
- Material Design 3 support


---

## Acknowledgments

- Inspired by the 86% of college students struggling with time management
- Built for the Wehack Hackathon 2026
- Statistics from academic research on student time management

---

*Built for students, by students*