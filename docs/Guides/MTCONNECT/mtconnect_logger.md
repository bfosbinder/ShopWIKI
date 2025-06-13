#!/usr/bin/env python3  asdasd
"""
MTConnect Data Logger - MySQL Storage
Collects MTConnect data via HTTP requests and stores in MySQL database
"""

import requestsasdasd
import mysql.connector
from mysql.connector import Error
import xml.etree.ElementTree as ET
from datetime import datetime
import time
import logging
import sys
from typing import Dict, List, Optional
import configparser

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('mtconnect_logger.log'),
        logging.StreamHandler(sys.stdout)
    ]
)
logger = logging.getLogger(__name__)

class MTConnectMySQLLogger:
    def __init__(self, config_file: str = 'config.ini'):
        """Initialize the MTConnect logger with configuration"""
        self.config = self.load_config(config_file)
        self.connection = None
        self.last_sequence = None
        
    def load_config(self, config_file: str) -> configparser.ConfigParser:
        """Load configuration from file"""
        config = configparser.ConfigParser()
        
        # Default configuration
        config['MTCONNECT'] = {
            'agent_url': 'http://localhost:5000',
            'device_name': '',
            'poll_interval': '5'
        }
        
        config['DATABASE'] = {
            'host': 'localhost',
            'port': '3306',
            'database': 'mtconnect_data',
            'user': 'mtconnect_user',
            'password': 'your_password'
        }
        
        # Try to read existing config file
        try:
            config.read(config_file)
            logger.info(f"Configuration loaded from {config_file}")
        except Exception as e:
            logger.warning(f"Could not read config file {config_file}, using defaults: {e}")
            
        return config
    
    def connect_database(self) -> bool:
        """Establish MySQL database connection"""
        try:
            self.connection = mysql.connector.connect(
                host=self.config['DATABASE']['host'],
                port=int(self.config['DATABASE']['port']),
                database=self.config['DATABASE']['database'],
                user=self.config['DATABASE']['user'],
                password=self.config['DATABASE']['password'],
                autocommit=True
            )
            
            if self.connection.is_connected():
                logger.info("Successfully connected to MySQL database")
                return True
                
        except Error as e:
            logger.error(f"Error connecting to MySQL: {e}")
            return False
    
    def create_tables(self):
        """Create necessary tables if they don't exist"""
        cursor = self.connection.cursor()
        
        # Create data_items table
        create_data_items_table = """
        CREATE TABLE IF NOT EXISTS data_items (
            id INT AUTO_INCREMENT PRIMARY KEY,
            timestamp DATETIME(6) NOT NULL,
            device_name VARCHAR(255) NOT NULL,
            data_item_id VARCHAR(255) NOT NULL,
            data_item_name VARCHAR(255),
            data_item_type VARCHAR(100),
            category VARCHAR(50),
            value TEXT,
            sequence_number BIGINT,
            INDEX idx_timestamp (timestamp),
            INDEX idx_device_data_item (device_name, data_item_id),
            INDEX idx_sequence (sequence_number)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
        """
        
        # Create events table for discrete events
        create_events_table = """
        CREATE TABLE IF NOT EXISTS events (
            id INT AUTO_INCREMENT PRIMARY KEY,
            timestamp DATETIME(6) NOT NULL,
            device_name VARCHAR(255) NOT NULL,
            data_item_id VARCHAR(255) NOT NULL,
            data_item_name VARCHAR(255),
            event_type VARCHAR(100),
            value TEXT,
            sequence_number BIGINT,
            INDEX idx_timestamp (timestamp),
            INDEX idx_device_event (device_name, data_item_id)
        ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
        """
        
        try:
            cursor.execute(create_data_items_table)
            cursor.execute(create_events_table)
            logger.info("Database tables created/verified successfully")
        except Error as e:
            logger.error(f"Error creating tables: {e}")
        finally:
            cursor.close()
    
    def get_mtconnect_data(self, endpoint: str = 'current') -> Optional[ET.Element]:
        """Fetch MTConnect data from agent"""
        agent_url = self.config['MTCONNECT']['agent_url']
        device_name = self.config['MTCONNECT']['device_name']
        
        # Build URL
        if device_name:
            url = f"{agent_url}/{device_name}/{endpoint}"
        else:
            url = f"{agent_url}/{endpoint}"
            
        # Add sequence parameter for streaming
        if endpoint == 'sample' and self.last_sequence:
            url += f"?from={self.last_sequence + 1}"
        
        try:
            logger.debug(f"Requesting: {url}")
            response = requests.get(url, timeout=30)
            response.raise_for_status()
            
            # Parse XML
            root = ET.fromstring(response.content)
            return root
            
        except requests.exceptions.RequestException as e:
            logger.error(f"HTTP request failed: {e}")
            return None
        except ET.ParseError as e:
            logger.error(f"XML parsing failed: {e}")
            return None
    
    def parse_and_store_data(self, xml_data: ET.Element):
        """Parse MTConnect XML and store in database"""
        if xml_data is None:
            return
            
        # Define namespaces
        namespaces = {
            'm': 'urn:mtconnect.org:MTConnectStreams:1.3',
            'mt': 'urn:mtconnect.org:MTConnectDevices:1.3'
        }
        
        cursor = self.connection.cursor()
        
        try:
            # Find all DeviceStream elements
            device_streams = xml_data.findall('.//m:DeviceStream', namespaces)
            
            for device_stream in device_streams:
                device_name = device_stream.get('name', 'Unknown')
                
                # Process ComponentStreams
                component_streams = device_stream.findall('.//m:ComponentStream', namespaces)
                
                for component_stream in component_streams:
                    component_name = component_stream.get('name', '')
                    
                    # Process all data items (Samples, Events, Condition)
                    for category in ['Samples', 'Events', 'Condition']:
                        category_element = component_stream.find(f'm:{category}', namespaces)
                        if category_element is not None:
                            self.process_category_data(cursor, category_element, device_name, component_name, category.lower())
            
            self.connection.commit()
            logger.debug("Data stored successfully")
            
        except Error as e:
            logger.error(f"Database error: {e}")
            self.connection.rollback()
        finally:
            cursor.close()
    
    def process_category_data(self, cursor, category_element: ET.Element, device_name: str, component_name: str, category: str):
        """Process data items within a category (samples, events, condition)"""
        
        for data_item in category_element:
            # Extract data item attributes
            data_item_id = data_item.get('dataItemId', '')
            timestamp_str = data_item.get('timestamp', '')
            sequence = data_item.get('sequence', 0)
            name = data_item.get('name', data_item.tag.split('}')[-1])  # Remove namespace
            
            # Convert timestamp
            try:
                if timestamp_str:
                    # MTConnect timestamp format: 2023-12-01T10:30:45.123Z
                    timestamp = datetime.fromisoformat(timestamp_str.replace('Z', '+00:00'))
                else:
                    timestamp = datetime.now()
            except ValueError:
                timestamp = datetime.now()
                logger.warning(f"Invalid timestamp format: {timestamp_str}")
            
            # Get value
            value = data_item.text if data_item.text else ''
            
            # Update last sequence number
            try:
                seq_num = int(sequence)
                if self.last_sequence is None or seq_num > self.last_sequence:
                    self.last_sequence = seq_num
            except (ValueError, TypeError):
                pass
            
            # Insert into appropriate table
            if category in ['samples', 'condition']:
                insert_query = """
                INSERT INTO data_items (timestamp, device_name, data_item_id, data_item_name, 
                                      data_item_type, category, value, sequence_number)
                VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
                """
                cursor.execute(insert_query, (
                    timestamp, device_name, data_item_id, name, 
                    data_item.tag.split('}')[-1], category, value, sequence
                ))
            else:  # events
                insert_query = """
                INSERT INTO events (timestamp, device_name, data_item_id, data_item_name, 
                                  event_type, value, sequence_number)
                VALUES (%s, %s, %s, %s, %s, %s, %s)
                """
                cursor.execute(insert_query, (
                    timestamp, device_name, data_item_id, name, 
                    data_item.tag.split('}')[-1], value, sequence
                ))
    
    def run_continuous_logging(self):
        """Run continuous data logging"""
        if not self.connect_database():
            logger.error("Failed to connect to database. Exiting.")
            return
        
        self.create_tables()
        
        poll_interval = int(self.config['MTCONNECT']['poll_interval'])
        logger.info(f"Starting continuous logging (poll interval: {poll_interval}s)")
        
        # Get initial current state
        logger.info("Getting initial current state...")
        current_data = self.get_mtconnect_data('current')
        if current_data is not None:
            self.parse_and_store_data(current_data)
        
        # Start continuous sampling
        while True:
            try:
                logger.debug("Fetching sample data...")
                sample_data = self.get_mtconnect_data('sample')
                if sample_data is not None:
                    self.parse_and_store_data(sample_data)
                
                time.sleep(poll_interval)
                
            except KeyboardInterrupt:
                logger.info("Received interrupt signal. Shutting down...")
                break
            except Exception as e:
                logger.error(f"Unexpected error in main loop: {e}")
                time.sleep(poll_interval)
        
        # Cleanup
        if self.connection and self.connection.is_connected():
            self.connection.close()
            logger.info("Database connection closed")

def create_sample_config():
    """Create a sample configuration file"""
    config = configparser.ConfigParser()
    
    config['MTCONNECT'] = {
        'agent_url': 'http://localhost:5000',
        'device_name': 'VMC-3Axis',
        'poll_interval': '5'
    }
    
    config['DATABASE'] = {
        'host': 'localhost',
        'port': '3306', 
        'database': 'mtconnect_data',
        'user': 'mtconnect_user',
        'password': 'your_password_here'
    }
    
    with open('config.ini', 'w') as configfile:
        config.write(configfile)
    
    print("Sample config.ini created. Please edit with your settings.")

if __name__ == "__main__":
    import argparse
    
    parser = argparse.ArgumentParser(description='MTConnect MySQL Data Logger')
    parser.add_argument('--create-config', action='store_true', 
                       help='Create sample configuration file')
    parser.add_argument('--config', default='config.ini',
                       help='Configuration file path (default: config.ini)')
    
    args = parser.parse_args()
    
    if args.create_config:
        create_sample_config()
    else:
        logger = MTConnectMySQLLogger(args.config)
        logger.run_continuous_logging()