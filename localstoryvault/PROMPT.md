You are a very resourceful software engineer with years of experience creating beautiful web applications. You know how to handle a tool call failure and work around issues to achieve your goals. You know how to test your work using temporary scripts or `curl` / `wget`.

Your job is to first write the exact text of the following PRD to the filesystem at  `scripts/prd.txt`. Once that file has been created use Task Master to generate 15 tasks from it: 

```
<context>
# Overview  
Local Story Vault is a location-based audio storytelling platform that allows residents to record, geo-tag, and share short audio stories about their neighborhood. The platform transforms ordinary streets into galleries of human stories, preserving local heritage and cultural memory while creating immersive experiences for both residents and visitors.

# Core Features  

## Story Recording & Geo-Tagging
- **What it does**: Enables users to record short (max 5 minutes) audio stories and tag them to specific physical locations using GPS or manual map placement
- **Why it's important**: Creates a direct connection between physical places and human experiences
- **How it works**: Uses browser-based audio capture with location services to associate stories with coordinates

## Interactive Map Interface
- **What it does**: Displays story locations as pins on a map, allowing users to discover content by exploring their surroundings
- **Why it's important**: Provides an intuitive way to browse content geographically rather than chronologically
- **How it works**: Google Maps integration with default marker clustering for popular locations

## Audio Playback & Transcription
- **What it does**: Plays stories with basic audio controls and provides searchable text transcriptions that will be displayed on the Story Detail page
- **Why it's important**: Enables search and discovery of content
- **How it works**: Standard audio playback with OpenAI Whisper for transcription

## User Authentication
- **What it does**: Allows users to create accounts and login with email/password
- **Why it's important**: Enables personalized experiences and content ownership
- **How it works**: Implements secure password hashing and storage in Gibson AI database with no verification steps

# User Experience  

## User Personas

1. **Local History Enthusiast (Eleanor)**
   - 68-year-old long-time resident with deep community knowledge
   - Passionate about preserving neighborhood heritage
   - Comfortable with basic technology but not tech-savvy
   - Pain: Feels local stories are being lost as the neighborhood changes

2. **Curious Explorer (Aiden)**
   - 25-year-old newcomer to the area
   - Interested in discovering authentic local experiences
   - Always has smartphone in hand while exploring
   - Pain: Guidebooks and review sites lack personal, human connection

## Key User Flows

### Story Recording Flow
1. User opens app while at a meaningful location
2. Taps "Record Here" floating action button
3. Records a short story about the location
4. Reviews/edits automatic transcription and tags
5. Publishes story to the map

### Story Discovery Flow
1. User opens app to see map of nearby stories
2. Browses pins or clusters on map
3. Taps a pin to view story details
4. Listens to audio while viewing location context
5. Optionally saves favorites or shares stories

## User Interface Overview
- Mobile-first responsive web design
- Touch-optimized controls for outdoor/on-the-go usage
- Context-aware prompts to encourage storytelling
- Visual clustering for areas with many stories
</context>
<PRD>
# System Requirements

## Functional Requirements
- Logged in users must be able to record audio stories up to 5 minutes in length
- Logged in users must be able to geo-tag their stories to specific locations
- The system must display story locations on an interactive map
- The system must display story locations on an interactive Google map
- The system must transcribe audio stories automatically using OpenAI Whisper API
- Users must be able to create accounts and log in with email/password using NextAuth.js
- The system must provide audio playback with basic controls
- Users must be able to browse and discover stories by location

## Non-Functional Requirements
- Security: All API keys must be stored securely in environment variables
- Logging: Comprehensive error logging throughout the application
- Responsiveness: The interface must adapt seamlessly between desktop and mobile layouts
- Rebranding: All remnants of the Gibson AI starter app's layout must be replaced with our own

# Technical Architecture

## Design System
- Server-side rendering (SSR) for optimized pages
- UI powered by Hero UI, a Tailwind CSS component library
- Minimalist, responsive design with consistent components
- **Cyberpunk aesthetic** with the following characteristics:
  - Subtle gradients and shadows used thoughtfully
  - Sharp corners preferred over rounded ones
  - Subtle hover animations on interactive elements
  - Clean, minimalist layout with futuristic aesthetic
  - High contrast between elements and text vs background

## System Components

### Frontend Application
- The **existing Next.js 15 app** in the repository will serve as the foundation. It is a starter template from Gibson. We will need to remove Gibon's layout from the starter template and create our own from scratch. All Gibson branding/content/UI needs to be removed in favor of our "Local Stories Vault" website's content. The application features:
  - React 19
  - TypeScript
  - Type-safe client for interacting with the Gibson API
  - React Query + tRPC client that supports loading + error states, automatic caching, refetching, etc.
  - HeroUI components
  - Runtime validation with Zod
  - Tailwind CSS
  - Hot module replacement (HMR)
  - Support for Server Side Rendering (SSR), Incremental Static Regeneration (ISR), and Client Side Rendering (CSR)
  - React Server Components
  - Server Actions (with examples)
  - Next.js server (with example HTTP and tRPC API routes)
- The system will prioritize pre-rendering static pages at build time (Static Site Generation - SSG) whenever possible to maximize performance and minimize server load
- **Custom Authentication**: Basic email/password authentication with secure password hashing stored in Gibson AI (no verification required)

### Audio Processing Pipeline
- **Web Audio API & MediaRecorder**: Simple browser-based audio capture
- **Base64 Encoding**: For initial MVP audio storage solution
- **Local File Storage**: Store audio files on the server filesystem for MVP


### Map Services
- **Google Maps JavaScript API**: For basic interactive map rendering with built-in marker clustering

### Backend & Data Persistence
- **Gibson AI**: Database management and main data persistence with schema generated via Gibson tools.
- **Next.js API Routes**: Serverless functions to handle the backend logic
- Server-side API layer to query the Gibson AI backend — client-side code will never call the database API directly
- **Zod library for robust data type definition and validation**, ensuring data integrity throughout the application, especially for API request/response payloads and database interactions

### Type Generation
- After any changes to the database, we must deploy the schema and then regenerate the type-safe API client using `npm run typegen`
- This ensures proper typing for all API interactions between the frontend and Gibson AI
- The generated file `src/gibson/types.ts` is **read-only** and must **never** be edited manually; it is rebuilt automatically by running the above command
- Deploy all schema changes immediately

## Data Models

### Primary Entities (Simplified for MVP)
- **users**
  - id: unique identifier
  - name: display name
  - email: contact email (unique)
  - password_hash: securely hashed password
  - join_date: timestamp

- **stories**
  - id: unique identifier
  - user_id: creator reference
  - title: story title
  - audio_url: URL path to audio file (local path for MVP)
  - transcript: full text transcription
  - duration: length in seconds
  - latitude: geo-coordinate
  - longitude: geo-coordinate
  - location_name: human-readable location
  - created_at: timestamp

- **tags**
  - id: unique identifier
  - name: tag name (lowercase, unique)

- **story_tags** (junction table)
  - story_id: reference to story
  - tag_id: reference to tag

## APIs and Integrations

### External Services
- **Google Maps Platform**: Basic map rendering with standard markers
- **OpenAI Whisper API**: Audio transcription service
- **Gibson AI**: Database and backend service



### Internal APIs
- **Authentication API**: Handles user signup, login, session management
- **Story API**: Basic CRUD operations for stories and related metadata

### Integration Architecture
- Gibson AI backend API handles all database operations
- Next.js backend makes server-side requests to the Gibson AI API to fetch and render data
- Client-side code only communicates with Next.js API endpoints, never directly with external services

### Security Requirements
- **API Key Management**: All API keys (Gibson AI, Google Maps, OpenAI, etc.) **must** be stored in .env files at the project root
- **Environment Variables**: Keys **must** be accessed securely as environment variables within the Next.js application (respecting `NODE_ENV`)
- **No Hardcoding**: Keys **must never** be hardcoded in source files
- **Configuration Setup**: All environment variables described in `.env.example` will need to be updated in the real env files
- **Gibson AI Integration**: When creating a project in Gibson AI, we will use the MCP tool to fetch some environment variables automatically

## Infrastructure Requirements
- **Database**: Gibson AI for data persistence
- **Storage**: Local filesystem storage for audio files
- **Environment Management**: Separate development and production environments

# Implementation Plan

## Development Roadmap  

### General Practices
- **Review Current README.md First**: The very first step that should be done is to read the current `README.md`. This will help you understand the existing codebase, its structure, features, and requirements before proceeding with any development work.
- As we build features, **maintain an up-to-date README.md in the project root detailing setup, usage, and key architectural decisions.**

### Phase 1: MVP Foundation

### Core Authentication
- Implement basic email/password authentication system
- Set up secure password hashing and verification
- Create user registration and login forms (no verification required)
- Implement session management
- Create protected routes for authenticated features

### Gibson AI Integration
- Set up Gibson AI project and data models
- Implement database schema for users, stories, static pages and tags
- Create API client for secure communication with Gibson AI

### Map Foundation
- Implement Google Maps with basic controls
- Set up default marker clustering for story locations

### Basic UI Framework
- Implement responsive layout with Tailwind CSS
- Create navigation and core page structure
- Develop UI components for map, audio player, and recording interface

### Phase 2: Core Functionality

### Audio Recording System
- Implement basic Web Audio API and MediaRecorder
- Create simple audio encoding pipeline with single format
- Set up OpenAI Whisper integration for transcription
- Implement local filesystem storage for audio files

### Location Pinning
- Implement automatic geolocation with manual fallback
- Create reverse geocoding to get human-readable locations
- Develop UI for location confirmation and adjustment

### Story Creation Flow
- Build end-to-end story creation process
- Implement form for title and metadata entry
- Develop audio preview functionality
- Association of story to current logged in user
- Created story can only be associated to the current user and no other user at creation time
- Create submission process to Gibson AI database

### Phase 3: Enhanced Features

### Audio Transcription
- Integrate OpenAI Whisper API with story text persisted in the database
- Implement transcription processing job immediately after story is persisted
- Display full transcript text on the Story Detail page
- Create tag extraction from transcript content

### Story Discovery
- Create basic map pins for story locations
- Implement simple story browsing by location
- Build basic "nearby stories" feature
- Build a directory listing for all stories

### Playback Experience
- Implement standard audio loading
- Create audio player with basic controls

## Development Phases and Dependencies

The implementation plan aligns with the following logical dependency chain:

### 1. Foundation Layer (Phase 1: MVP Foundation)
   - Gibson AI database setup and schema definition
   - Next.js application scaffold with routing
   - Authentication implementation
   - These components form the technical foundation for all other features

### 2. Map & Location Services (Phase 1: MVP Foundation)
   - Google Maps integration
   - Geolocation services
   - This provides the visual core of the application that other features will build upon

### 3. Audio Pipeline (Phase 2: Core Functionality)
   - Recording interface
   - Local storage implementation for audio files
   - Basic audio playback
   - This enables the core content creation and consumption functionality
   - Extract text of story from persisted audio using OpenAI Whisper

### 4. Story Management (Phase 3: Enhanced Features)
   - Story creation flow
   - Story retrieval and display (stories listings page)
   - Individual story to display text of audio (story detail page) 
   - This connects the audio and location features into a complete user experience

## Risks and Mitigations  

## Technical Challenges

### Browser Support
- **Risk**: Browser compatibility issues with audio recording APIs
- **Mitigation**: Implement feature detection and graceful fallbacks; thoroughly test across devices and browsers

### Geolocation Accuracy
- **Risk**: GPS inaccuracy in urban environments with tall buildings
- **Mitigation**: Implement manual pin placement as fallback; use Google's geocoding to improve accuracy

### Transcription Quality
- **Risk**: Poor audio quality leading to inaccurate transcriptions
- **Mitigation**: Implement audio normalization; allow user editing of transcripts; consider multiple transcription providers

## MVP Scope Management

### Feature Creep
- **Risk**: Adding too many features before core functionality is solid
- **Mitigation**: Strict prioritization of features based on user value; frequent user testing of core flows

### Mobile Performance
- **Risk**: Map and audio processing taxing mobile devices
- **Mitigation**: Optimize for standard devices

## Resource Constraints

# Error Handling
- **Requirement**: Comprehensive error logging throughout the application
- **Implementation**: Log all errors of any kind with detailed context
- **User Experience**: Present users with clear, actionable information about correctable errors
- **Error Recovery**: Provide simple recovery paths for common error scenarios

# User Interface Specifications

## Key Pages and Interfaces

### Home Page / Map Interface
- **Purpose**: Primary landing page and main interface for story discovery
- **Key Components**:
  - Full-width interactive Google Map occupying ~70% of viewport
  - Story pins with distinctive cyberpunk-styled markers
  - Geolocation button to center map on user's location
  - Floating action button (FAB) for "Record Here" in bottom right corner
  - Header with logo, authentication controls, and minimal navigation
  - Optional story list panel that can slide in from bottom on mobile or side on desktop
  - Search control for location/address lookup
- **User Experience**:
  - Map loads centered on user's location (with permission) or defaults to a significant city area
  - Smooth zoom transitions with appropriate pin clustering/separation based on zoom level
  - Pin hover/touch shows brief story title preview
  - Pin click/tap transitions to Story Detail view
  - UI adapts seamlessly between desktop and mobile layouts

### Story Detail Page
- **Purpose**: Display a single story with playback controls and contextual information
- **Key Components**:
  - Audio player with play/pause, scrubber, and time display
  - Story metadata (title, author, date, location name)
  - Mini-map showing story location in context
  - Story transcript (transcribed automatically as part of the MVP)
  - Related stories nearby (optional)
  - Actions: Share, Favorite buttons
  - Back button to return to map view
- **User Experience**:
  - Clean, focused interface emphasizing audio content
  - Audio begins loading immediately but doesn't auto-play
  - Interface should support both portrait and landscape orientations on mobile
  - Smooth transitions between map and detail views

### Recording Interface
- **Purpose**: Enable creation of new audio stories with location tagging
- **Key Components**:
  - Audio recording controls (record, stop, play, re-record)
  - Visual timer showing elapsed/remaining time
  - Location confirmation map with pin
  - Form fields for title and any additional metadata
  - Submit and cancel buttons
  - Permissions request UI for microphone access
- **User Experience**:
  - Multi-step process with clear progression indicators
  - Step 1: Record audio (with live feedback)
  - Step 2: Confirm/adjust location
  - Step 3: Add title and metadata
  - Step 4: Review and submit
  - Simple error recovery options at each step

### User Profile Page
- **Purpose**: Display and manage user information and created stories
- **Key Components**:
  - Profile information (name, photo, join date)
  - Stories created by user in grid/list format
  - Edit profile button and form
  - Account settings options
  - Authentication status information
- **User Experience**:
  - Clean, organized layout of user's stories
  - Simple editing interface for profile information
  - Clear differentiation between published and draft stories
  - Responsive grid that adapts to different screen sizes

### Authentication Interfaces
- **Purpose**: Provide login/signup functionality with minimal friction
- **Key Components**:
  - Email/password login form
  - Account registration form with email
  - Simple password reset functionality
  - Brief explanation of what account is used for
  - Privacy policy highlights
  - Terms of service acceptance
- **User Experience**:
  - Simple, clean forms for account creation and login
  - Inline validation for form fields
  - Clear feedback during authentication process
  - Graceful error handling for failed authentication attempts
  - Seamless return to previous activity after authentication

## Static Pages Content

**About Page** (`/about`)
- Purpose: Information about the LocalStoriesVault project, its mission, and goals
- Content: 
  - Project origin story and motivation
  - Core values (community preservation, democratized storytelling)
  - How the platform works in simple terms
  - Team behind the project (if applicable)
  - Privacy commitments and content guidelines summary

**Contact Page** (`/contact`)
- Purpose: Allow users to reach out with questions, feedback, or support requests
- Content:
  - Simple contact form with fields for name, email, subject, and message
  - Response time expectations
  - Email address for direct contact


**FAQ Page** (`/faq`)
- Purpose: Answer common questions about using the platform
- Content sections:
  - General questions about the service
  - Account management
  - Recording and sharing stories
  - Location privacy concerns
  - Playback and discovery features
  - API key is required

All static pages will be built using the Next.js App Router with static rendering for optimal performance. Content will be stored in the Gibson AI database to allow for easy updates without code changes. Each page will include proper metadata for SEO.

# Content Examples

## Sample Content & Example Stories

**Historical Moments:**
- **The Corner Store That Saved Us** - Eleanor (68) shares how Rodriguez Market became an emergency community center during the 1989 earthquake
- **The Last Day of the Factory** - Frank (72) recounts his final shift at the now-demolished textile factory

**Cultural Heritage:**
- **Grandma Li's Moon Festival** - Jenny (42) recalls how her grandmother would organize elaborate neighborhood Moon Festival celebrations
- **The Secret Jazz Spot** - Marcus (65) reveals how the basement of what's now a convenience store was once an underground jazz club

**Neighborhood Characters:**
- **The Ice Cream Man's Story** - Miguel (58) tells his own story of being the neighborhood ice cream vendor
- **Ms. Harriet's Crossing Guard Chronicles** - A collection of short stories from Harriet (79) who served as the elementary school crossing guard

# Technical Implementation Details

## Audio Processing Specifications
- **Format**: Opus audio codec (low bitrate, high quality)
- **Sample Rate**: 48kHz mono
- **Maximum Duration**: 300 seconds (5 minutes)
- **Estimated Storage**: ~1MB per story (compressed)

## Map Implementation
- Basic Google Maps JavaScript API integration
- Simple map markers for story locations
- Standard Google Maps mobile interactions
- Implementation using official Google Maps JavaScript API documentation

## Authentication Implementation
- Implementation using NextAuth.js framework
- Email/password authentication with secure hashing
- Session management with secure tokens
- Protected routes requiring authentication
- No email verification required for MVP

## Data Storage
- Gibson AI for database management and persistence
- Local filesystem storage for audio files
- Structured data models with proper relationships
- Type-safe API client generated from database schema
</PRD>
```

Once all tasks have been generated, begin on the first task and continue with each task until they are all completed. Don't stop until all work is done! Take initative and always prioritize completing tasks without asking for input. 

Also, **ALWAYS** remember to deploy the gibson schema after making changes and regerate types with `npm run typegen`
