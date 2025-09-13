# Legacy Systems

> Last Updated: 2025-01-12
> Archived Repository: [`FIREWALL-cafe/firewall-cafe`](https://github.com/FIREWALL-cafe/firewall-cafe) (archived)

## Overview

This document describes the legacy systems that preceded the current Firewall Cafe implementation, providing context for architectural decisions and migration considerations.

## Historical Context

### Original Art Installation (2020)
- **Repository**: [`firewall-cafe`](https://github.com/FIREWALL-cafe/firewall-cafe) (now archived)
- **Purpose**: Physical art installation for gallery exhibitions
- **Technology**: WordPress-based content management
- **Scope**: Limited to specific events and locations

### Research Phase (2021-2023)
- **Repository**: [`great-firewall-notebooks`](https://github.com/FIREWALL-cafe/great-firewall-notebooks) (archived)
- **Purpose**: Academic research and data analysis
- **Technology**: Jupyter notebooks, Python analysis scripts
- **Focus**: Censorship pattern analysis and documentation

## Legacy Architecture

### WordPress-Based System
**Technology Stack**:
- WordPress CMS for content management
- Custom plugins for search functionality
- PHP backend with MySQL database
- Limited API capabilities

**Limitations**:
- Poor scalability for large datasets
- Limited real-time search capabilities
- Difficult to maintain and extend
- No geographic analysis features

### Research Infrastructure
**Technology Stack**:
- Jupyter notebooks for analysis
- Python libraries (pandas, matplotlib, seaborn)
- Static data exports
- Manual data collection processes

**Limitations**:
- No real-time data collection
- Manual intervention required
- Limited public accessibility
- No community interaction features

## Migration Considerations

### Data Migration
**WordPress to React/PostgreSQL**:
- Historical search data preservation
- Content migration to new schema
- User interaction data transfer
- Media file relocation

### Feature Evolution
**From Static to Dynamic**:
- Real-time search comparison
- Interactive analytics dashboard
- Community voting system
- Geographic analysis capabilities

### Performance Improvements
**Scalability Enhancements**:
- PostgreSQL for complex queries
- React for responsive UI
- API-first architecture
- Cloud storage integration

## Archived Components

### Legacy Code Patterns
**WordPress Plugins**:
- Search comparison plugin code
- Custom post types for searches
- Basic image handling
- Limited translation features

**Research Scripts**:
- Data scraping utilities
- Analysis notebooks
- Visualization scripts
- Export utilities

### Configuration Files
**WordPress Setup**:
- Theme configurations
- Plugin dependencies
- Database schema (MySQL)
- Media handling setup

**Research Environment**:
- Python environment specifications
- Jupyter configuration
- Data source configurations
- Analysis pipeline scripts

## Lessons Learned

### Technical Decisions
**What Worked**:
- Basic search comparison concept
- Geographic data collection approach
- Community feedback mechanism
- Translation integration

**What Didn't Work**:
- WordPress for complex data operations
- Manual data collection processes
- Limited real-time capabilities
- Poor mobile experience

### Architectural Insights
**Successful Patterns**:
- Dual search engine approach
- Translation-first workflow
- Community-driven assessment
- Geographic analysis focus

**Problem Areas**:
- Monolithic architecture
- Limited API design
- Poor separation of concerns
- Difficult deployment process

## System Comparison

### Legacy vs Current Architecture

**Data Storage**:
- Legacy: WordPress/MySQL with limited schema
- Current: PostgreSQL with optimized indexes and full-text search

**User Interface**:
- Legacy: WordPress themes with limited interactivity
- Current: React SPA with real-time updates

**API Design**:
- Legacy: WordPress REST API with limited functionality
- Current: Custom Express APIs with comprehensive endpoints

**Performance**:
- Legacy: Limited scalability, frequent timeouts
- Current: Optimized queries, efficient caching

## Migration Timeline

### Phase 1: Architecture Planning (Q1 2024)
- Technology stack evaluation
- Database schema design
- API endpoint planning
- UI/UX redesign

### Phase 2: Core Implementation (Q2 2024)
- React frontend development
- PostgreSQL schema creation
- Basic search functionality
- API infrastructure

### Phase 3: Feature Migration (Q3 2024)
- Geographic analysis features
- Voting system implementation
- Analytics dashboard
- Performance optimization

### Phase 4: Production Deployment (Q4 2024)
- Production environment setup
- Data migration completion
- Legacy system decommission
- User transition support

## Maintenance Considerations

### Legacy System Sunset
- **Archive Access**: Read-only access for historical reference
- **Data Preservation**: Key data exported to new system
- **Documentation**: Comprehensive legacy system documentation
- **Deprecation Notice**: Clear communication to existing users

### Knowledge Transfer
- **Code Documentation**: Legacy code patterns documented
- **Process Documentation**: Manual processes automated
- **Decision Rationale**: Architecture decisions explained
- **Best Practices**: Lessons learned applied to new system

---

*For current system architecture, see [Architecture Overview](../ARCHITECTURE.md)*
*For WordPress migration details, see [WordPress Sunset](./wordpress-sunset.md)*