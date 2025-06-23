# Airalo-API-automation
Automate requests to the Airalo Partner API and validate the responses
#!/usr/bin/env python3
"""
Airalo Partner API Test Automation
QA Core Network Coding Exercise - Task 1

This script automates API requests to the Airalo Partner API and validates responses.
It performs OAuth2 authentication, places eSIM orders, and verifies eSIM lists.

Author: QA Engineer
Date: June 2025
"""

import requests
import json
import time
import sys
from typing import Dict, Any, Optional, List
import logging

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('airalo_api_test.log'),
        logging.StreamHandler(sys.stdout)
    ]
)
logger = logging.getLogger(__name__)

class AiraloAPITester:
    """
    Airalo Partner API Test Automation Class
    Handles OAuth2 authentication, order placement, and eSIM list verification
    """
    
    def __init__(self):
        # API Configuration
        self.base_url = "https://sandbox-partners-api.airalo.com"
        self.client_id = "7e29e2facf83359855f746fc490443e6"
        self.client_secret = "e5NNajm6jNAzrWsKoAdr41WfDiMeS1l6IcGdhmbb"
        self.access_token = None
        self.token_expires_at = None
        
        # Test Configuration
        self.package_slug = "merhaba-7days-1gb"
        self.order_quantity = 6
        self.order_id = None
        
        # Session for connection reuse
        self.session = requests.Session()
        self.session.headers.update({
            'Content-Type': 'application/json',
            'Accept': 'application/json'
        })
    
    def authenticate(self) -> bool:
        """
        Obtain OAuth2 token using client credentials grant
        Returns: True if authentication successful, False otherwise
        """
        logger.info("Starting OAuth2 authentication...")
        
        auth_url = f"{self.base_url}/v2/token"
        auth_data = {
            'grant_type': 'client_credentials',
            'client_id': self.client_id,
            'client_secret': self.client_secret,
            'scope': 'read write'
        }
        
        try:
            response = self.session.post(
                auth_url,
                data=auth_data,
                headers={'Content-Type': 'application/x-www-form-urlencoded'}
            )
            
            logger.info(f"Authentication response status: {response.status_code}")
            
            if response.status_code == 200:
                token_data = response.json()
                self.access_token = token_data.get('access_token')
                expires_in = token_data.get('expires_in', 3600)
                self.token_expires_at = time.time() + expires_in
                
                # Update session headers with bearer token
                self.session.headers.update({
                    'Authorization': f'Bearer {self.access_token}'
                })
                
                logger.info("OAuth2 authentication successful")
                return True
            else:
                logger.error(f"Authentication failed: {response.status_code} - {response.text}")
                return False
                
        except requests.exceptions.RequestException as e:
            logger.error(f"Authentication request failed: {str(e)}")
            return False
    
    def is_token_valid(self) -> bool:
        """
        Check if current token is still valid
        Returns: True if token is valid, False otherwise
        """
        if not self.access_token or not self.token_expires_at:
            return False
        
        # Check if token expires in next 60 seconds
        return time.time() < (self.token_expires_at - 60)
    
    def ensure_authenticated(self) -> bool:
        """
        Ensure we have a valid token, refresh if necessary
        Returns: True if authenticated, False otherwise
        """
        if not self.is_token_valid():
            return self.authenticate()
        return True
    
    def place_esim_order(self) -> Dict[str, Any]:
        """
        Place an order for eSIMs using the specified package
        Returns: API response data
        """
        logger.info(f"Placing order for {self.order_quantity} {self.package_slug} eSIMs...")
        
        if not self.ensure_authenticated():
            raise Exception("Authentication failed")
        
        order_url = f"{self.base_url}/v2/orders"
        order_data = {
            "quantity": self.order_quantity,
            "package_id": self.package_slug,
            "type": "esim",
            "description": f"Test order for {self.order_quantity} {self.package_slug} eSIMs"
        }
        
        try:
            response = self.session.post(order_url, json=order_data)
            logger.info(f"Order response status: {response.status_code}")
            
            response_data = response.json() if response.content else {}
            
            if response.status_code == 201:
                self.order_id = response_data.get('data', {}).get('id')
                logger.info(f"Order placed successfully. Order ID: {self.order_id}")
            else:
                logger.error(f"Order placement failed: {response.status_code} - {response.text}")
            
            return {
                'status_code': response.status_code,
                'response_data': response_data,
                'success': response.status_code == 201
            }
            
        except requests.exceptions.RequestException as e:
            logger.error(f"Order request failed: {str(e)}")
            return {
                'status_code': 0,
                'response_data': {'error': str(e)},
                'success': False
            }
    
    def get_esim_list(self, limit: int = 50, page: int = 1) -> Dict[str, Any]:
        """
        Retrieve list of eSIMs
        Args:
            limit: Number of records per page
            page: Page number
        Returns: API response data
        """
        logger.info("Retrieving eSIM list...")
        
        if not self.ensure_authenticated():
            raise Exception("Authentication failed")
        
        esim_url = f"{self.base_url}/v2/sims"
        params = {
            'limit': limit,
            'page': page
        }
        
        try:
            response = self.session.get(esim_url, params=params)
            logger.info(f"eSIM list response status: {response.status_code}")
            
            response_data = response.json() if response.content else {}
            
            if response.status_code == 200:
                esim_count = len(response_data.get('data', []))
                logger.info(f"Retrieved {esim_count} eSIMs")
            else:
                logger.error(f"eSIM list retrieval failed: {response.status_code} - {response.text}")
            
            return {
                'status_code': response.status_code,
                'response_data': response_data,
                'success': response.status_code == 200
            }
            
        except requests.exceptions.RequestException as e:
            logger.error(f"eSIM list request failed: {str(e)}")
            return {
                'status_code': 0,
                'response_data': {'error': str(e)},
                'success': False
            }
    
    def validate_order_response(self, order_result: Dict[str, Any]) -> List[str]:
        """
        Validate order placement response
        Args:
            order_result: Result from place_esim_order method
        Returns: List of validation errors (empty if all validations pass)
        """
        errors = []
        
        # Validate status code
        if order_result['status_code'] != 201:
            errors.append(f"Expected status code 201, got {order_result['status_code']}")
        
        # Validate response structure
        response_data = order_result.get('response_data', {})
        
        if not response_data.get('data'):
            errors.append("Response missing 'data' field")
        else:
            order_data = response_data['data']
            
            # Validate order ID
            if not order_data.get('id'):
                errors.append("Order response missing 'id' field")
            
            # Validate quantity if present
            if 'quantity' in order_data and order_data['quantity'] != self.order_quantity:
                errors.append(f"Expected quantity {self.order_quantity}, got {order_data['quantity']}")
            
            # Validate package information
            if 'package_id' in order_data and order_data['package_id'] != self.package_slug:
                errors.append(f"Expected package_id {self.package_slug}, got {order_data['package_id']}")
        
        return errors
    
    def validate_esim_list_response(self, esim_result: Dict[str, Any]) -> List[str]:
        """
        Validate eSIM list response
        Args:
            esim_result: Result from get_esim_list method
        Returns: List of validation errors (empty if all validations pass)
        """
        errors = []
        
        # Validate status code
        if esim_result['status_code'] != 200:
            errors.append(f"Expected status code 200, got {esim_result['status_code']}")
        
        # Validate response structure
        response_data = esim_result.get('response_data', {})
        
        if not isinstance(response_data.get('data'), list):
            errors.append("Response 'data' field should be a list")
            return errors
        
        esim_list = response_data['data']
        
        # Count eSIMs with the target package slug
        target_esims = [
            esim for esim in esim_list 
            if esim.get('package_id') == self.package_slug or 
               esim.get('package', {}).get('slug') == self.package_slug
        ]
        
        # Validate count
        if len(target_esims) < self.order_quantity:
            errors.append(
                f"Expected at least {self.order_quantity} eSIMs with package '{self.package_slug}', "
                f"found {len(target_esims)}"
            )
        
        # Validate eSIM properties
        for i, esim in enumerate(target_esims[:self.order_quantity]):
            if not esim.get('iccid'):
                errors.append(f"eSIM {i+1} missing ICCID")
            
            if not esim.get('qr_code') and not esim.get('qr_code_url'):
                errors.append(f"eSIM {i+1} missing QR code information")
        
        return errors
    
    def run_test_suite(self) -> Dict[str, Any]:
        """
        Run the complete test suite
        Returns: Test results summary
        """
        logger.info("Starting Airalo Partner API Test Suite...")
        
        results = {
            'authentication': {'success': False, 'errors': []},
            'order_placement': {'success': False, 'errors': []},
            'esim_list_retrieval': {'success': False, 'errors': []},
            'overall_success': False
        }
        
        try:
            # Step 1: Authentication
            if self.authenticate():
                results['authentication']['success'] = True
                logger.info("✓ Authentication successful")
            else:
                results['authentication']['errors'].append("Failed to authenticate")
                logger.error("✗ Authentication failed")
                return results
            
            # Step 2: Place eSIM Order
            order_result = self.place_esim_order()
            order_errors = self.validate_order_response(order_result)
            
            if order_result['success'] and not order_errors:
                results['order_placement']['success'] = True
                logger.info("✓ Order placement successful")
            else:
                results['order_placement']['errors'] = order_errors
                logger.error(f"✗ Order placement failed: {order_errors}")
            
            # Wait a moment for order processing
            time.sleep(2)
            
            # Step 3: Retrieve and Validate eSIM List
            esim_result = self.get_esim_list()
            esim_errors = self.validate_esim_list_response(esim_result)
            
            if esim_result['success'] and not esim_errors:
                results['esim_list_retrieval']['success'] = True
                logger.info("✓ eSIM list retrieval and validation successful")
            else:
                results['esim_list_retrieval']['errors'] = esim_errors
                logger.error(f"✗ eSIM list validation failed: {esim_errors}")
            
            # Overall success
            results['overall_success'] = (
                results['authentication']['success'] and
                results['order_placement']['success'] and
                results['esim_list_retrieval']['success']
            )
            
        except Exception as e:
            logger.error(f"Test suite execution failed: {str(e)}")
            results['execution_error'] = str(e)
        
        return results
    
    def print_test_summary(self, results: Dict[str, Any]) -> None:
        """
        Print a formatted test summary
        Args:
            results: Test results from run_test_suite
        """
        print("\n" + "="*60)
        print("AIRALO PARTNER API TEST RESULTS")
        print("="*60)
        
        print(f"Authentication: {'PASS' if results['authentication']['success'] else 'FAIL'}")
        if results['authentication']['errors']:
            for error in results['authentication']['errors']:
                print(f"  - {error}")
        
        print(f"Order Placement: {'PASS' if results['order_placement']['success'] else 'FAIL'}")
        if results['order_placement']['errors']:
            for error in results['order_placement']['errors']:
                print(f"  - {error}")
        
        print(f"eSIM List Validation: {'PASS' if results['esim_list_retrieval']['success'] else 'FAIL'}")
        if results['esim_list_retrieval']['errors']:
            for error in results['esim_list_retrieval']['errors']:
                print(f"  - {error}")
        
        print(f"\nOVERALL RESULT: {'PASS' if results['overall_success'] else 'FAIL'}")
        print("="*60)

def main():
    """
    Main execution function
    """
    tester = AiraloAPITester()
    results = tester.run_test_suite()
    tester.print_test_summary(results)
    
    # Save results to file
    with open('test_results.json', 'w') as f:
        json.dump(results, f, indent=2)
    
    # Exit with appropriate code
    sys.exit(0 if results['overall_success'] else 1)

if __name__ == "__main__":
    main()
