# Forex Trading System

> A high-performance, modular algorithmic trading system for Forex markets built in modern C++ with focus on low-latency execution and statistical strategies.

[![C++ Standard](https://img.shields.io/badge/C++-20-blue.svg)](https://en.cppreference.com/w/cpp/20)
[![CMake](https://img.shields.io/badge/CMake-3.20+-green.svg)](https://cmake.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)]()

## Disclaimer

**This is educational software for learning algorithmic trading concepts.** 

- Trading financial instruments involves substantial risk of loss
- Past performance does not guarantee future results
- This software is provided "as is" without warranty of any kind
- The authors are not responsible for any financial losses
- Always test thoroughly on demo accounts before risking real capital

## Project Goals

### Phase 1: Foundation (Current)
- Build high-performance C++ trading infrastructure
- Implement statistical mean reversion strategy
- Develop event-driven backtesting framework
- Deploy on IC Markets demo account
- **Target Latency: 1-10ms execution**
- **Target Performance: Break-even while learning**

### Phase 2: Optimization (Future)
- Multiple strategy portfolio
- Lock-free data structures
- SIMD optimizations
- FIX protocol integration
- Co-location deployment

## Architecture and folders structure 

```
forex-trading-system/
├── include/                    # Public headers
│   ├── core/
│   │   ├── engine.hpp         # Main trading engine
│   │   ├── event.hpp          # Event system
│   │   └── time_utils.hpp     # High-resolution timing
│   ├── data/
│   │   ├── market_data.hpp    # Market data structures
│   │   ├── tick_handler.hpp   # Real-time tick processing
│   │   └── database.hpp       # TimescaleDB interface
│   ├── strategies/
│   │   ├── base_strategy.hpp  # Strategy interface
│   │   ├── mean_reversion.hpp # Statistical mean reversion
│   │   └── signals.hpp        # Signal generation
│   ├── execution/
│   │   ├── order_manager.hpp  # Order lifecycle
│   │   ├── broker_adapter.hpp # Broker API interface
│   │   └── fill_simulator.hpp # Realistic fill simulation
│   ├── risk/
│   │   ├── position_manager.hpp
│   │   ├── risk_limits.hpp
│   │   └── circuit_breaker.hpp
│   └── utils/
│       ├── logger.hpp         # Fast logging (spdlog)
│       ├── config.hpp         # YAML configuration
│       └── metrics.hpp        # Performance metrics
├── src/                       # Implementation files
│   ├── core/
│   ├── data/
│   ├── strategies/
│   ├── execution/
│   ├── risk/
│   └── utils/
├── tests/                     # Unit tests (Google Test)
│   ├── unit/
│   ├── integration/
│   └── performance/
├── python/                    # Python bindings (pybind11)
│   ├── bindings/
│   └── analysis/              # Post-trade analysis
├── benchmarks/                # Performance benchmarks
├── config/                    # Configuration files
│   ├── strategies.yaml
│   ├── risk_limits.yaml
│   └── credentials.yaml.example
├── scripts/                   # Build & deployment scripts
├── docker/                    # Docker configuration
├── docs/                      # Documentation
├── CMakeLists.txt            # Main CMake file
└── conanfile.txt             # Dependencies (Conan)
```

##  Quick Start

### Prerequisites

- **Compiler**: GCC 11+, Clang 14+, or MSVC 2022+
- **CMake**: 3.20 or higher
- **Conan**: 2.0+ (package manager)
- **PostgreSQL**: 14+ with TimescaleDB
- **Redis**: 7+
- **IC Markets**: Demo account with API access

### System Requirements
- **OS**: Linux (Ubuntu 22.04+ recommended), macOS 12+, or Windows 11
- **RAM**: 8GB minimum, 16GB recommended
- **CPU**: Multi-core processor (4+ cores recommended)

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/yourusername/forex-trading-system.git
cd forex-trading-system
```

2. **Install dependencies with Conan**
```bash
# Install Conan if not already installed
pip install conan

# Create default profile
conan profile detect --force

# Install dependencies
conan install . --output-folder=build --build=missing
```

3. **Build the project**
```bash
# Configure with CMake
cmake -B build -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_TOOLCHAIN_FILE=build/conan_toolchain.cmake

# Build
cmake --build build --config Release -j$(nproc)

# Run tests
cd build && ctest --output-on-failure
```

4. **Set up configuration**
```bash
cp config/credentials.yaml.example config/credentials.yaml
# Edit config/credentials.yaml with your IC Markets demo credentials
```

5. **Run data collector**
```bash
./build/bin/data_collector --mode demo --config config/strategies.yaml
```

6. **Run backtest**
```bash
./build/bin/backtester \
  --strategy mean_reversion \
  --start-date 2024-01-01 \
  --end-date 2024-12-01 \
  --config config/strategies.yaml
```

### Docker Setup (Recommended for Development)

```bash
# Build Docker image
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f trading-system

# Run tests in container
docker-compose exec trading-system ctest

# Stop services
docker-compose down
```

## Initial Strategy: Statistical Mean Reversion

### Concept
Identifies and trades temporary price deviations in correlated currency pairs using z-score analysis.

### Implementation
```cpp
class MeanReversionStrategy : public BaseStrategy {
public:
    Signal generateSignal(const MarketData& data) override;
    void onFill(const Fill& fill) override;
    
private:
    double calculateZScore(const PriceSeries& spread);
    double calculateHedgeRatio(const PriceSeries& pair1, 
                               const PriceSeries& pair2);
};
```

### Key Parameters
- **Pairs**: EUR/USD & GBP/USD
- **Lookback**: 20-period rolling window
- **Entry**: Z-score > ±2.0
- **Exit**: Z-score returns to 0 or stop-loss
- **Timeframe**: 5-minute bars
- **Position Size**: 2% risk per trade

### Performance Targets (Backtest)
- Sharpe Ratio: 1.2 - 1.8
- Max Drawdown: < 15%
- Win Rate: 55-65%
- Execution Latency: < 5ms average

##  Development

### Building in Debug Mode
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

### Running Tests
```bash
# Run all tests
cd build && ctest

# Run specific test suite
./build/tests/strategy_tests

# Run with verbose output
ctest -V

# Run performance benchmarks
./build/benchmarks/latency_benchmark
```

### Code Quality Tools
```bash
# Format code (clang-format)
find src include -name "*.cpp" -o -name "*.hpp" | xargs clang-format -i

# Static analysis (clang-tidy)
clang-tidy src/**/*.cpp -- -Iinclude

# Memory leak detection (Valgrind)
valgrind --leak-check=full ./build/bin/backtester
```

### Performance Profiling
```bash
# CPU profiling with perf (Linux)
perf record -g ./build/bin/trading_engine
perf report

# Memory profiling with Valgrind
valgrind --tool=massif ./build/bin/trading_engine
ms_print massif.out.*
```

## 📈 Key Dependencies

Managed via Conan (`conanfile.txt`):

```ini
[requires]
boost/1.83.0
spdlog/1.12.0          # Fast logging
fmt/10.1.1             # String formatting
yaml-cpp/0.8.0         # Configuration
libpq/15.4             # PostgreSQL client
hiredis/1.2.0          # Redis client
nlohmann_json/3.11.2   # JSON parsing
websocketpp/0.8.2      # WebSocket client
openssl/3.1.3          # TLS/SSL
pybind11/2.11.1        # Python bindings
gtest/1.14.0           # Testing framework

[generators]
CMakeToolchain
CMakeDeps
```

## Configuration

### Risk Management (`config/risk_limits.yaml`)
```yaml
max_position_size: 0.02        # 2% of account per trade
max_daily_loss: 0.05           # 5% daily loss limit
max_drawdown: 0.15             # 15% max drawdown
max_open_positions: 3
max_correlation: 0.7
stop_loss_pct: 0.015           # 1.5% stop loss
take_profit_pct: 0.025         # 2.5% take profit
```

### Strategy Parameters (`config/strategies.yaml`)
```yaml
mean_reversion:
  enabled: true
  symbol_pairs:
    - pair1: "EURUSD"
      pair2: "GBPUSD"
  z_score_entry: 2.0
  z_score_exit: 0.0
  lookback_period: 20
  min_correlation: 0.85
  execution:
    max_latency_ms: 10
    retry_attempts: 3
    timeout_ms: 5000
```

## 📊 Monitoring & Logging

### Logging System (spdlog)
```cpp
// Example: High-performance logging
logger->info("Order filled: {} @ {} lots={}", 
             symbol, price, lots);
logger->warn("High latency detected: {}ms", latency);
logger->error("Connection lost to broker");
```

### Metrics Collection
- Order execution latency (microsecond precision)
- Strategy P&L (real-time)
- Risk metrics (exposure, drawdown)
- System health (CPU, memory, network)

### Grafana Dashboard
Access at `http://localhost:3000`

**Real-time metrics:**
- Tick-to-trade latency histogram
- Order book depth visualization
- Position P&L tracking
- System resource utilization

## 🔧 Advanced Features

### Low-Latency Optimizations
```cpp
// Cache-friendly data structures
struct alignas(64) OrderBookLevel {
    double price;
    double volume;
    uint64_t timestamp;
};

// Lock-free order queue
boost::lockfree::queue<Order> order_queue{1000};

// Memory pool for order allocation
boost::pool<> order_pool{sizeof(Order)};
```

### SIMD Price Calculations
```cpp
// AVX2 optimized z-score calculation
__m256d calculate_zscore_simd(const double* prices, 
                               size_t size);
```

## 📚 Documentation

- [Build Guide](docs/build/README.md)
- [Architecture Overview](docs/architecture/design.md)
- [Strategy Development](docs/strategies/development_guide.md)
- [Performance Tuning](docs/performance/optimization.md)
- [API Reference](docs/api/README.md) (Doxygen)

## 🗺️ Roadmap

### Q1 2025
- [x] CMake build system
- [x] Core event-driven architecture
- [ ] Basic mean reversion strategy
- [ ] Backtesting engine with realistic fills
- [ ] Risk management framework
- [ ] Unit test coverage > 80%

### Q2 2025
- [ ] IC Markets FIX protocol integration
- [ ] Paper trading on demo account
- [ ] Real-time monitoring dashboard
- [ ] Performance optimization (SIMD, lock-free)
- [ ] Strategy parameter optimization

### Q3 2025
- [ ] Multi-strategy portfolio management
- [ ] Machine learning signal pipeline
- [ ] Advanced order types (iceberg, TWAP)
- [ ] Live trading (if profitable on demo)

### Q4 2025
- [ ] FPGA acceleration research
- [ ] Alternative data integration
- [ ] Multi-broker support
- [ ] Advanced risk analytics

##  Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md).

### Code Style
- Follow [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
- Use `clang-format` with provided `.clang-format`
- Document public APIs with Doxygen comments
- Write unit tests for new features

### Pull Request Process
1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Write tests (aim for >80% coverage)
4. Ensure all tests pass (`ctest`)
5. Format code (`clang-format -i **/*.cpp **/*.hpp`)
6. Commit changes with clear messages
7. Push to branch
8. Open Pull Request with description

## 📄 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file.

## 🙏 Acknowledgments

- IC Markets for demo trading environment
- [QuantLib](https://www.quantlib.org/) for financial calculations
- Open-source C++ community

## 📞 Contact

- **Issues**: [GitHub Issues](https://github.com/yourusername/forex-trading-system/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/forex-trading-system/discussions)

## 📈 Performance Benchmarks

| Metric | Target | Actual |
|--------|--------|--------|
| Order Latency (p50) | < 5ms | TBD |
| Order Latency (p99) | < 10ms | TBD |
| Throughput | > 1000 orders/sec | TBD |
| Memory Usage | < 500MB | TBD |
| CPU Usage (avg) | < 30% | TBD |

## 🚀 Getting Started with Git

```bash
# Initialize repository
git init

# Create README
# (Copy the content from this artifact to README.md)

# Stage all files
git add .

# First commit
git commit -m "Initial commit: Project structure and documentation"

# Create main branch
git branch -M main

# Add remote (replace with your GitHub repo URL)
git remote add origin https://github.com/yourusername/forex-trading-system.git

# Push to GitHub
git push -u origin main
```

---

**Remember**: Start with correctness, then optimize for performance. Test thoroughly before deploying capital.