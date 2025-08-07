# Personal Document Library MCP Server

A Model Context Protocol (MCP) server that enables Claude to access and analyze a personal collection of documents through RAG (Retrieval-Augmented Generation). The system processes PDFs, Word documents, and EPUBs locally, creates semantic search capabilities, and provides synthesis across multiple sources.

## Features

- **9 Powerful Tools** for Claude integration
- **Local RAG System** with ChromaDB vector storage
- **Semantic Search** with synthesis capabilities  
- **Background Monitoring** with automatic indexing
- **Automatic PDF Cleaning** for problematic files
- **Web Dashboard** for real-time monitoring at http://localhost:8888
- **Lazy Initialization** for fast MCP startup
- **ARM64 Compatible** (Apple Silicon optimized)
- **Multi-format Support** (PDF, DOCX, DOC, PPTX, PPT, EPUB, TXT)

## Quick Start

### Prerequisites

- **macOS** (tested on macOS 14+)
- **Python 3.9+** (3.11+ recommended)
- **Claude Desktop** installed
- **Homebrew** (for package management)
- **~4GB RAM** for embeddings

### Optional Dependencies (Auto-installed by setup)

- **ocrmypdf** - For OCR processing of scanned PDFs
- **LibreOffice** - For processing Word documents (.doc, .docx)
- **pandoc** - For EPUB file processing

### Installation

#### Option 1: Interactive Setup (Recommended)

```bash
# Clone the repository
git clone <your-repo-url>
cd AITools

# Run interactive setup - will prompt for your books directory
./quick_start.sh
```

The interactive setup will:
- ✅ Check Python installation
- ✅ Create virtual environment
- ✅ Install all dependencies
- ✅ Ask for your books directory location
- ✅ Generate configuration files
- ✅ Install background services (optional)
- ✅ Start web monitoring dashboard (optional)
- ✅ Run initial indexing (optional)

#### Option 2: Automated Setup

```bash
# Clone the repository
git clone <your-repo-url>
cd AITools

# Run automated setup with your books path
./setup.sh --books-path /path/to/your/books --install-service --start-web-monitor
```

Available options:
- `--books-path PATH` - Path to your document library
- `--db-path PATH` - Path for vector database (default: ./chroma_db)
- `--non-interactive` - Run without prompts
- `--install-service` - Install background indexing service
- `--start-web-monitor` - Start web dashboard

### Post-Installation

#### 1. Configure Claude Desktop

Copy the generated configuration to Claude Desktop:

```bash
# macOS
cp config/claude_desktop_config.json ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Restart Claude Desktop for changes to take effect
```

#### 2. Access Web Dashboard

Open http://localhost:8888 in your browser to:
- 📊 View indexing progress
- 📚 Browse indexed documents
- 🔍 Search your library
- ⚙️ Monitor system status

## Services

### Background Services (macOS)

The system includes two LaunchAgent services that run automatically:

1. **Index Monitor Service** - Watches for new documents and indexes them
2. **Web Monitor Service** - Provides the web dashboard

#### Service Management

```bash
# Check service status
./scripts/service_status.sh
./scripts/webmonitor_service_status.sh

# Install services
./scripts/install_service.sh          # Index monitor
./scripts/install_webmonitor_service.sh # Web monitor

# Uninstall services
./scripts/uninstall_service.sh
./scripts/uninstall_webmonitor_service.sh

# View logs
tail -f logs/index_monitor_stdout.log
tail -f logs/webmonitor_stdout.log
```

## Usage

### Running the MCP Server

```bash
# Start MCP server (for Claude Desktop)
./scripts/run.sh

# Index documents only
./scripts/run.sh --index-only

# Index with retry for large collections
./scripts/run.sh --index-only --retry
```

### Manual Operations

```bash
# Start/stop web monitor manually
./scripts/start_web_monitor.sh
./scripts/stop_web_monitor.sh

# Check indexing status
./scripts/indexing_status.sh

# Monitor indexing progress continuously
watch -n 5 "./scripts/indexing_status.sh"

# Manage failed documents
./scripts/manage_failed_docs.sh list     # View failed documents
./scripts/manage_failed_docs.sh add      # Add document to skip list
./scripts/manage_failed_docs.sh remove   # Remove from skip list
./scripts/manage_failed_docs.sh retry    # Clear list to retry all
./scripts/cleanup_failed_list.sh         # Remove successfully indexed docs from failed list

# Pause/resume indexing
./scripts/pause_indexing.sh
./scripts/resume_indexing.sh
```

## Configuration

### Environment Variables

The system uses environment variables for configuration. These can be set in your shell or in the `.env` file:

```bash
# Books directory (where your PDFs/documents are stored)
export PERSONAL_LIBRARY_DOC_PATH="/path/to/your/books"

# Database directory (for vector storage)
export PERSONAL_LIBRARY_DB_PATH="/path/to/database"

# Logs directory
export PERSONAL_LIBRARY_LOGS_PATH="/path/to/logs"
```

### Directory Structure

```
AITools/
├── books/              # Your document library (configurable)
├── chroma_db/          # Vector database storage
├── logs/               # Application logs
├── config/             # Configuration files
├── scripts/            # Utility scripts
├── src/                # Source code
│   ├── core/          # Core RAG functionality
│   ├── servers/       # MCP server implementation
│   ├── indexing/      # Document indexing
│   └── monitoring/    # Web dashboard
└── venv_mcp/          # Python virtual environment
```

## Tools Available in Claude

Once configured, Claude will have access to these tools:

1. **search_books** - Semantic search across your library
2. **get_book_content** - Retrieve specific document content
3. **list_books** - Browse available documents
4. **synthesize_sources** - Compare multiple sources on a topic
5. **find_related_content** - Discover related passages
6. **analyze_themes** - Extract themes from documents
7. **get_book_metadata** - Get document information
8. **search_by_category** - Search within categories
9. **create_summary** - Generate document summaries

## Troubleshooting

### Common Issues

#### Indexer Gets Stuck on Large/Corrupted PDFs
```bash
# Check which file is stuck
cat chroma_db/index_status.json

# Use the failed docs manager script
./scripts/manage_failed_docs.sh list                    # View all failed documents
./scripts/manage_failed_docs.sh add "path/to/file.pdf"  # Add to skip list
./scripts/manage_failed_docs.sh remove "file.pdf"       # Remove from skip list
./scripts/manage_failed_docs.sh retry                   # Clear list to retry all

# Or manually add to failed list
echo '{"path/to/file.pdf": {"error": "Manual skip", "cleaned": false}}' >> chroma_db/failed_pdfs.json

# Restart the service
./scripts/uninstall_service.sh
./scripts/install_service.sh
```

#### Missing Dependencies for Document Processing
```bash
# Check and install OCR support for scanned PDFs
which ocrmypdf || brew install ocrmypdf

# Check and install LibreOffice for Word documents
which soffice || brew install --cask libreoffice

# Check and install pandoc for EPUB files
which pandoc || brew install pandoc

# Verify installations
ocrmypdf --version
soffice --version
pandoc --version
```

#### "Too Many Open Files" Errors
```bash
# Check current limit
ulimit -n

# Increase file descriptor limit (temporary)
ulimit -n 4096

# For permanent fix on macOS, add to ~/.zshrc or ~/.bash_profile:
echo "ulimit -n 4096" >> ~/.zshrc

# Restart the indexing service
./scripts/uninstall_service.sh
./scripts/install_service.sh
```

#### Service Keeps Restarting
```bash
# Check service logs for crashes
tail -f logs/index_monitor_stderr.log

# Check for lock files
ls -la /tmp/spiritual_library_index.lock

# Remove stale lock (if older than 30 minutes)
rm /tmp/spiritual_library_index.lock

# Monitor service health
./scripts/service_status.sh
watch -n 5 "./scripts/service_status.sh"
```

#### Web Monitor Not Accessible
```bash
# Check if service is running
launchctl list | grep webmonitor

# Restart the service
./scripts/uninstall_webmonitor_service.sh
./scripts/install_webmonitor_service.sh

# Check logs
tail -f logs/webmonitor_stdout.log

# Verify port is not in use
lsof -i :8888
```

#### Indexing Not Working
```bash
# Check service status
./scripts/service_status.sh

# View error logs
tail -f logs/index_monitor_stderr.log

# Check indexing progress
./scripts/indexing_status.sh

# Manually reindex
./scripts/run.sh --index-only

# Reindex with retry for large collections
./scripts/run.sh --index-only --retry
```

#### Permission Issues
```bash
# Fix permissions for scripts
chmod +x scripts/*.sh

# Fix Python symlinks
./setup.sh --non-interactive

# Fix directory permissions
chmod -R 755 logs/
chmod -R 755 chroma_db/
```

#### Word Documents Not Processing
```bash
# Verify LibreOffice is installed
which soffice || brew install --cask libreoffice

# Test LibreOffice manually
soffice --headless --convert-to pdf test.docx

# Check for temporary lock files (start with ~$)
find books/ -name "~\$*" -delete
```

### Reset and Clean

If you need to start fresh:

```bash
# Remove vector database
rm -rf chroma_db/*

# Uninstall services
./scripts/uninstall_service.sh
./scripts/uninstall_webmonitor_service.sh

# Reinstall
./quick_start.sh
```

## Document Support

### Supported Formats
- **PDF** (.pdf) - Including scanned PDFs with OCR
- **Word** (.docx, .doc) - Requires LibreOffice
- **PowerPoint** (.pptx, .ppt) - Requires LibreOffice
- **EPUB** (.epub) - Requires pandoc
- **Text** (.txt) - Plain text files

### Installing Optional Dependencies

For full document support:

```bash
# For Word documents
brew install --cask libreoffice

# For EPUB files
brew install pandoc

# For better PDF handling
brew install ghostscript
```

## Development

### Running Tests
```bash
# Activate virtual environment
source venv_mcp/bin/activate

# Run tests (when available)
python -m pytest tests/
```

### Adding New Document Types

Edit `src/core/shared_rag.py` to add support for new formats:
1. Add file extension to `SUPPORTED_EXTENSIONS`
2. Implement loader in `load_document()`
3. Update categorization if needed

## Security Notes

- All processing is done locally - no data leaves your machine
- Database is stored locally in `chroma_db/`
- Services run with user permissions only
- No network access required except for web dashboard

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Test your changes
4. Submit a pull request

## License

[Your License Here]

## Support

For issues or questions:
- Open an issue on GitHub
- Check logs in `logs/` directory
- Review `CLAUDE.md` for development details
- See `QUICK_REFERENCE.md` for command reference