# Airalo Partner API Test Automation Framework

## Overview

This repository contains a comprehensive test automation framework for the Airalo Partner API, designed to validate eSIM ordering and management workflows. 
The framework implements OAuth2 authentication, automated API testing, and detailed response validation.

## Objectives

- **Automate API Requests**: Streamline interactions with Airalo Partner API endpoints
- **Validate Responses**: Comprehensive assertion framework for API response verification  
- **OAuth2 Integration**: Secure authentication with automatic token management
- **Detailed Reporting**: Extensive logging and test result reporting

## Architecture

### Core Components

1. **AiraloAPIClient**: Handles API communication and authentication
2. **AiraloAPITestSuite**: Comprehensive test execution and validation
3. **TestResults**: Structured result tracking and reporting
4. **APICredentials**: Secure credential management

### Test Cases Implemented

#### Test Case 1: eSIM Order Placement
- **Endpoint**: `POST /v2/orders`
- **Action**: Place order for 6 "merhaba-7days-1gb" eSIMs
- **Validations**:
  - HTTP 200 status code verification
  - Response structure validation
  - Order quantity confirmation

#### Test Case 2: eSIM List Validation  
- **Endpoint**: `GET /v2/sims`
- **Action**: Retrieve and validate eSIM inventory
- **Validations**:
  - HTTP 200 status code verification
  - Minimum 6 eSIMs present
  - Package slug verification ("merhaba-7days-1gb")
  - Response data structure validation

## Prerequisites

### System Requirements
- **Python**: 3.7 or higher
- **Operating System**: Windows, macOS, or Linux
- **Internet Connection**: Required for API access

### Python Dependencies
```bash
requests>=2.28.0
dataclasses (built-in for Python 3.7+)
logging (built-in)
datetime (built-in)
json (built-in)
time (built-in)
typing (built-in)
```

## Installation & Setup

### 1. Clone Repository
```bash
git clone <repository-url>
cd airalo-api-tests
```

### 2. Create Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv airalo-env

# Activate virtual environment
# On Windows:
airalo-env\Scripts\activate
# On macOS/Linux:
source airalo-env/bin/activate
```

### 3. Install Dependencies
```bash
pip install requests
# OR using requirements.txt
pip install -r requirements.txt
```

### 4. Verify Installation
```bash
python --version  # Should be 3.7+
python -c "import requests; print('Dependencies installed successfully')"
```

## Usage Instructions

### Basic Execution
```bash
# Run complete test suite
python airalo_api_tests.py
```

### Advanced Usage

#### Custom Logging Level
```bash
# Set logging level via environment variable
export LOG_LEVEL=DEBUG
python airalo_api_tests.py
```

#### Individual Test Execution
```python
# Example: Run only order placement test
from airalo_api_tests import AiraloAPITestSuite

suite = AiraloAPITestSuite()
result = suite.test_esim_order_placement()
print(f"Test Status: {result.status}")
```

##  Output & Reporting

### Console Output
The framework provides real-time console output with:
- Test execution progress
- Assertion results with ✓/✗ indicators  
- Response times and status codes
- Detailed error messages
- Comprehensive summary report

### Log Files
- **File**: `airalo_api_tests.log`
- **Content**: Complete execution history with timestamps
- **Retention**: Appends to existing log file

### Sample Output
```
2024-12-18 10:30:15 - INFO - ================================================================================
2024-12-18 10:30:15 - INFO - AIRALO PARTNER API TEST SUITE EXECUTION  
2024-12-18 10:30:15 - INFO - ================================================================================
2024-12-18 10:30:15 - INFO - Authenticating with Airalo Partner API...
2024-12-18 10:30:16 - INFO - Authentication response: 200 in 0.85s
2024-12-18 10:30:16 - INFO - Authentication successful

==================================================
Starting Test: eSIM Order Placement
==================================================
2024-12-18 10:30:16 - INFO - Placing order for 6 merhaba-7days-1gb eSIMs...
2024-12-18 10:30:17 - INFO - Order response: 200 in 1.23s
2024-12-18 10:30:17 - INFO - ✓ eSIM Order Placement: Status code assertion passed (200)
2024-12-18 10:30:17 - INFO - ✓ eSIM Order Placement: Response contains key 'data'
2024-12-18 10:30:17 - INFO - ✓ eSIM Order Placement: Order quantity matches (6)
```

## Configuration

### API Credentials
Credentials are embedded in the `APICredentials` dataclass:
```python
@dataclass
class APICredentials:
    client_id: str = "7e29e2facf83359855f746fc490443e6"
    client_secret: str = "e5NNajm6jNAzrWsKoAdr41WfDiMeS1l6IcGdhmbb"
    base_url: str = "https://partners-api.airalo.com"
```

### Customizable Parameters
- **eSIM Package**: Default "merhaba-7days-1gb"
- **Order Quantity**: Default 6 eSIMs
- **API Timeout**: Configurable via requests session
- **Retry Logic**: Built-in authentication retry

## Test Implementation Details

### Authentication Flow
1. **OAuth2 Token Request**: Client credentials grant type
2. **Token Validation**: Automatic expiration checking
3. **Auto-Renewal**: Seamless token refresh when needed
4. **Secure Headers**: Bearer token authentication

### Request Handling
- **Session Management**: Persistent HTTP connections
- **Error Handling**: Comprehensive exception management
- **Response Timing**: Accurate performance measurement
- **Status Validation**: HTTP status code verification

### Assertion Framework
```python
# Example assertion implementation
def assert_status_code(self, response, expected, test_name):
    actual = response.status_code
    if actual == expected:
        logger.info(f"✓ {test_name}: Status code assertion passed ({actual})")
        return True
    else:
        logger.error(f"✗ {test_name}: Expected {expected}, Got {actual}")
        return False
```

##  Troubleshooting

### Common Issues

#### Authentication Failures
**Error**: `Authentication failed: 401 - Unauthorized`
**Solution**: 
- Verify client credentials are correct
- Check network connectivity
- Ensure API endpoints are accessible

#### Network Connectivity Issues  
**Error**: `Authentication request failed: Connection timeout`
**Solution**:
- Check internet connection
- Verify firewall settings
- Test API endpoint accessibility: `curl https://partners-api.airalo.com/v2/token`

#### Package Slug Validation Failures
**Error**: `Not all eSIMs have expected package slug`
**Solution**:
- Verify "merhaba-7days-1gb" package exists
- Check if order was processed correctly
- Review eSIM inventory via manual API call

### Debug Mode
Enable detailed debugging:
```python
import logging
logging.getLogger().setLevel(logging.DEBUG)
```

### Manual API Testing
Test individual endpoints using curl:
```bash
# Get OAuth token
curl -X POST https://partners-api.airalo.com/v2/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials&client_id=7e29e2facf83359855f746fc490443e6&client_secret=e5NNajm6jNAzrWsKoAdr41WfDiMeS1l6IcGdhmbb"

# Place order (replace TOKEN with actual token)
curl -X POST https://partners-api.airalo.com/v2/orders \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"quantity":6,"package_id":"merhaba-7days-1gb","type":"sim"}'
```

## Performance Metrics

### Expected Response Times
- **Authentication**: < 2 seconds
- **Order Placement**: < 3 seconds  
- **eSIM List Retrieval**: < 2 seconds

### Test Execution Time
- **Complete Suite**: ~10-15 seconds
- **Individual Tests**: ~5-8 seconds each

## Security Considerations

### Credential Management
- Credentials embedded for exercise purposes
- Production implementation should use environment variables
- Consider secrets management systems for enterprise use

### API Rate Limiting
- Framework includes delays between test executions
- Implements session reuse for efficiency
- Respects API rate limits

## Future Enhancements

### Planned Features
- **Parameterized Testing**: Support multiple package types
- **Data-Driven Tests**: CSV/JSON test data input
- **Parallel Execution**: Concurrent test execution
- **HTML Reporting**: Visual test reports
- **CI/CD Integration**: Jenkins/GitHub Actions support

### Extension Points
```python
# Example extension for additional test cases
class ExtendedAiraloTestSuite(AiraloAPITestSuite):
    def test_esim_cancellation(self):
        # Implementation for order cancellation testing
        pass
    
    def test_usage_monitoring(self):
        # Implementation for eSIM usage validation
        pass
```

## Support & Maintenance

### Issue Reporting
For issues or enhancements:
1. Check existing logs for error details
2. Verify network connectivity and credentials
3. Review troubleshooting section
4. Submit detailed issue report with logs

### Code Quality
- **Linting**: PEP 8 compliance
- **Documentation**: Comprehensive docstrings
- **Testing**: Self-validating test framework
- **Logging**: Detailed execution tracking
