# WordPress Plugin Collection

## Overview
This collection includes three WordPress plugins that demonstrate different aspects of plugin development:
1. Enhanced WooCommerce Checkout
2. Custom Jobs Manager 
3. API Posts Display Integration

## 1. Enhanced WooCommerce Checkout

## Introduction

The Enhanced WooCommerce Checkout plugin extends the standard WooCommerce checkout process by adding a crucial feature: delivery instructions. This enhancement allows customers to provide specific delivery guidance, improving the overall delivery experience and reducing potential miscommunication between customers and delivery personnel.

## Implementation Details

Let's examine the complete implementation with detailed explanations of each component:

```php
<?php
/**
 * Plugin Name: Custom Checkout Fields
 * Description: Adds a "Delivery Instructions" field to WooCommerce checkout.
 * Version: 1.0
 * Author: CodeFire Test
 */

// Security measure: Prevent direct file access
if ( ! defined( 'ABSPATH' ) ) {
    exit; // Prevent direct access
}

/**
 * Adds the delivery instructions field to the checkout page
 * This function hooks into the WooCommerce checkout form after the order notes
 * 
 * @param WC_Checkout $checkout The checkout object containing form data
 */
add_action( 'woocommerce_after_order_notes', 'add_delivery_instructions_field' );
function add_delivery_instructions_field( $checkout ) {
    // Create a container for the field with a clear heading
    echo '' . __('Delivery Instructions') . '';

    // Generate the form field using WooCommerce's built-in form field generator
    woocommerce_form_field( 'delivery_instructions', array(
        'type'        => 'textarea',           // Field type: multi-line text area
        'class'       => array( 'form-row-wide' ), // Full-width field
        'label'       => __('Any specific instructions for delivery?'),
        'placeholder' => __('e.g., Leave package at the front door.'),
    ), $checkout->get_value( 'delivery_instructions' )); // Pre-populate with any existing value

    echo '';
}

/**
 * Saves the delivery instructions when an order is placed
 * This function stores the instructions as order meta data
 * 
 * @param int $order_id The ID of the order being processed
 */
add_action( 'woocommerce_checkout_update_order_meta', 'save_delivery_instructions' );
function save_delivery_instructions( $order_id ) {
    // Only save if instructions were provided
    if ( ! empty( $_POST['delivery_instructions'] ) ) {
        // Sanitize the input and save it as order meta data
        update_post_meta( 
            $order_id, 
            '_delivery_instructions', // Meta key prefixed with underscore to hide in custom fields
            sanitize_textarea_field( $_POST['delivery_instructions'] ) 
        );
    }
}

/**
 * Displays the delivery instructions in the admin order page
 * This function adds the instructions to the shipping address section
 * 
 * @param WC_Order $order The order object being displayed
 */
add_action( 'woocommerce_admin_order_data_after_shipping_address', 'display_delivery_instructions_admin', 10, 1 );
function display_delivery_instructions_admin( $order ) {
    // Retrieve the saved instructions
    $delivery_instructions = get_post_meta( $order->get_id(), '_delivery_instructions', true );
    
    // Display instructions if they exist
    if ( ! empty( $delivery_instructions ) ) {
        echo '' . __('Delivery Instructions') . ': ' . 
             esc_html( $delivery_instructions ) . '';
    }
}
```

## Understanding the Components

Let's break down each major component of the plugin to understand how it works:

### Security Implementation

The plugin begins with a crucial security check:
```php
if ( ! defined( 'ABSPATH' ) ) {
    exit;
}
```
This prevents direct access to the plugin file, ensuring it can only be executed within the WordPress environment. This is a fundamental security practice that protects against potential exploitation.

### Field Addition Process

The checkout field addition is handled through WordPress's action hook system. The plugin uses `woocommerce_after_order_notes` to place the field in a logical location after the standard order notes. The implementation follows these steps:

1. Creates a containing div with a unique ID for potential styling
2. Uses WooCommerce's form field generator for consistency
3. Implements a textarea for multi-line input
4. Provides helpful placeholder text to guide users

### Data Storage Implementation

The plugin implements a robust data storage system using WordPress's post meta functionality:

1. Hooks into `woocommerce_checkout_update_order_meta` to capture the submission
2. Verifies data existence before processing
3. Sanitizes input using WordPress's built-in functions
4. Stores data with a prefixed meta key for organization

### Admin Display Integration

The admin-side display is seamlessly integrated into the WooCommerce order view:

1. Uses the `woocommerce_admin_order_data_after_shipping_address` hook
2. Retrieves stored instructions using get_post_meta
3. Implements proper escaping for secure output
4. Maintains consistent styling with WooCommerce's admin interface

## Installation Guide

1. Upload the plugin files to your WordPress installation
2. Activate the plugin through the WordPress plugins page
3. The delivery instructions field will automatically appear on your checkout page

## Usage Instructions

### For Customers
Customers will find a new field on the checkout page where they can:
1. Enter specific delivery instructions
2. Provide multiple lines of text if needed
3. See helpful placeholder text for guidance

### For Store Administrators
Store administrators can:
1. View delivery instructions in the order details
2. Find instructions prominently displayed with shipping information
3. Access instructions when processing orders



## 2.Custom Jobs Manager Plugin

## Overview

The Custom Jobs Manager is a WordPress plugin that creates a comprehensive job listing management system. It implements a custom post type for jobs with additional metadata fields, an administrative interface, and a frontend display system using shortcodes.

## Implementation Details

### Core Plugin Structure
```php
<?php
/**
 * Plugin Name: Custom Jobs Manager
 * Description: A plugin to manage job listings with custom fields
 * Version: 1.0.0
 * Author: CodeFireSolution Test
 * Text Domain: custom-jobs-manager
 */

if (!defined('ABSPATH')) {
    exit;
}

class CustomJobsManager {
    private static $instance = null;

    public static function getInstance() {
        if (self::$instance == null) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    private function __construct() {
        // Register hooks
        add_action('init', array($this, 'registerCustomPostType'));
        add_action('add_meta_boxes', array($this, 'addCustomMetaBoxes'));
        add_action('save_post', array($this, 'saveCustomMetaData'));
        add_action('admin_menu', array($this, 'addSettingsPage'));
        add_action('admin_init', array($this, 'registerSettings'));
        add_shortcode('job_listings', array($this, 'jobListingsShortcode'));
    }

    public function registerCustomPostType() {
        $labels = array(
            'name'               => __('Jobs', 'custom-jobs-manager'),
            'singular_name'      => __('Job', 'custom-jobs-manager'),
            'menu_name'          => __('Jobs', 'custom-jobs-manager'),
            'add_new'           => __('Add New Job', 'custom-jobs-manager'),
            'add_new_item'      => __('Add New Job', 'custom-jobs-manager'),
            'edit_item'         => __('Edit Job', 'custom-jobs-manager'),
            'view_item'         => __('View Job', 'custom-jobs-manager'),
            'all_items'         => __('All Jobs', 'custom-jobs-manager'),
            'search_items'      => __('Search Jobs', 'custom-jobs-manager'),
        );

        $args = array(
            'labels'              => $labels,
            'public'              => true,
            'has_archive'         => true,
            'publicly_queryable'  => true,
            'show_ui'             => true,
            'show_in_menu'        => true,
            'menu_icon'           => 'dashicons-businessman',
            'supports'            => array('title', 'editor'),
            'rewrite'             => array('slug' => 'jobs'),
        );

        register_post_type('job', $args);
    }

    public function addCustomMetaBoxes() {
        add_meta_box(
            'job_details',
            __('Job Details', 'custom-jobs-manager'),
            array($this, 'renderMetaBox'),
            'job',
            'normal',
            'high'
        );
    }

    public function renderMetaBox($post) {
        wp_nonce_field('job_details_nonce', 'job_details_nonce');
        
        $company_name = get_post_meta($post->ID, '_company_name', true);
        $location = get_post_meta($post->ID, '_location', true);
        $salary = get_post_meta($post->ID, '_salary', true);
        
        if (empty($salary)) {
            $salary = get_option('default_job_salary', '');
        }
        ?>
        <p>
            <label for="company_name"><?php _e('Company Name:', 'custom-jobs-manager'); ?></label><br>
            <input type="text" id="company_name" name="company_name" value="<?php echo esc_attr($company_name); ?>" size="50">
        </p>
        <p>
            <label for="location"><?php _e('Location:', 'custom-jobs-manager'); ?></label><br>
            <input type="text" id="location" name="location" value="<?php echo esc_attr($location); ?>" size="50">
        </p>
        <p>
            <label for="salary"><?php _e('Salary:', 'custom-jobs-manager'); ?></label><br>
            <input type="text" id="salary" name="salary" value="<?php echo esc_attr($salary); ?>" size="50">
        </p>
        <?php
    }

    public function saveCustomMetaData($post_id) {
        if (!isset($_POST['job_details_nonce']) || !wp_verify_nonce($_POST['job_details_nonce'], 'job_details_nonce')) {
            return;
        }

        if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) {
            return;
        }

        if (!current_user_can('edit_post', $post_id)) {
            return;
        }

        $fields = array('company_name', 'location', 'salary');
        
        foreach ($fields as $field) {
            if (isset($_POST[$field])) {
                update_post_meta($post_id, '_' . $field, sanitize_text_field($_POST[$field]));
            }
        }
    }

    public function addSettingsPage() {
        add_submenu_page(
            'edit.php?post_type=job',
            __('Jobs Settings', 'custom-jobs-manager'),
            __('Settings', 'custom-jobs-manager'),
            'manage_options',
            'job-settings',
            array($this, 'renderSettingsPage')
        );
    }

    public function registerSettings() {
        register_setting('job_settings', 'default_job_salary');
    }

    public function renderSettingsPage() {
        ?>
        <div class="wrap">
            <h1><?php _e('Jobs Settings', 'custom-jobs-manager'); ?></h1>
            <form method="post" action="options.php">
                <?php
                settings_fields('job_settings');
                do_settings_sections('job_settings');
                ?>
                <table class="form-table">
                    <tr>
                        <th scope="row">
                            <label for="default_job_salary"><?php _e('Default Salary', 'custom-jobs-manager'); ?></label>
                        </th>
                        <td>
                            <input type="text" id="default_job_salary" name="default_job_salary" 
                                value="<?php echo esc_attr(get_option('default_job_salary')); ?>" class="regular-text">
                        </td>
                    </tr>
                </table>
                <?php submit_button(); ?>
            </form>
        </div>
        <?php
    }

    public function jobListingsShortcode($atts) {
        $args = array(
            'post_type' => 'job',
            'posts_per_page' => -1,
            'post_status' => 'publish'
        );

        $query = new WP_Query($args);
        
        ob_start();
        
        if ($query->have_posts()) {
            echo '<div class="job-listings">';
            while ($query->have_posts()) {
                $query->the_post();
                $company_name = get_post_meta(get_the_ID(), '_company_name', true);
                $location = get_post_meta(get_the_ID(), '_location', true);
                $salary = get_post_meta(get_the_ID(), '_salary', true);
                ?>
                <div class="job-listing">
                    <h3><?php the_title(); ?></h3>
                    <p><strong><?php _e('Company:', 'custom-jobs-manager'); ?></strong> <?php echo esc_html($company_name); ?></p>
                    <p><strong><?php _e('Location:', 'custom-jobs-manager'); ?></strong> <?php echo esc_html($location); ?></p>
                    <p><strong><?php _e('Salary:', 'custom-jobs-manager'); ?></strong> <?php echo esc_html($salary); ?></p>
                    <div class="job-description">
                        <?php the_content(); ?>
                    </div>
                    <a href="<?php the_permalink(); ?>" class="read-more"><?php _e('View Details', 'custom-jobs-manager'); ?></a>
                </div>
                <?php
            }
            echo '</div>';
        } else {
            echo '<p>' . __('No jobs found.', 'custom-jobs-manager') . '</p>';
        }
        
        wp_reset_postdata();
        
        return ob_get_clean();
    }
}

// Initialize the plugin
add_action('plugins_loaded', array('CustomJobsManager', 'getInstance'));

// Activation Hook
register_activation_hook(__FILE__, 'custom_jobs_manager_activate');
function custom_jobs_manager_activate() {
    // Trigger the registration of the CPT
    CustomJobsManager::getInstance()->registerCustomPostType();
    // Flush rewrite rules
    flush_rewrite_rules();
}

// Deactivation Hook
register_deactivation_hook(__FILE__, 'custom_jobs_manager_deactivate');
function custom_jobs_manager_deactivate() {
    // Flush rewrite rules
    flush_rewrite_rules();
}
```

## Key Features Explained


### Custom Post Type
The plugin creates a custom post type 'job' with a complete set of labels and arguments. The post type supports titles and content editor by default, and uses a business-themed dashicon for the admin menu.

### Custom Meta Fields
Three custom meta fields are implemented for each job posting:
1. Company Name: Stores the employer's company name
2. Location: Captures the job location
3. Salary: Records the compensation information with a default value option

### Settings Page
A dedicated settings page is added as a submenu under the Jobs menu in the WordPress admin. It currently manages:
- Default salary setting that pre-populates the salary field for new job posts

### Frontend Display
The plugin includes a shortcode `[job_listings]` that displays job listings with:
- Job title
- Company information
- Location
- Salary
- Full job description
- A link to the detailed view

### Security Features
The implementation includes several security measures:
1. ABSPATH check to prevent direct file access
2. Nonce verification for form submissions
3. Capability checks for user permissions
4. Data sanitization for all saved fields
5. Proper escaping for displayed data

## Installation and Usage

1. Upload the plugin to your WordPress installation
2. Activate the plugin through the WordPress admin interface
3. Configure default settings under Jobs > Settings
4. Create job listings using the new Jobs menu item
5. Display job listings on any page using the shortcode `[job_listings]`

### Adding a Job Listing
1. Navigate to Jobs > Add New in the WordPress admin
2. Fill in the job title and description
3. Complete the Job Details meta box with company information
4. Publish the job posting

### Displaying Job Listings
Add the shortcode to any page or post:
```
[job_listings]
```



## 3. API Posts Display

## Introduction

The API Posts Display plugin creates an elegant way to showcase external content from the JSONPlaceholder API within WordPress. It implements caching for performance, error handling for reliability, and presents posts in a modern card-based layout. Let's explore how each component works together to create this functionality.

## Complete Implementation

```php
<?php
/**
 * Plugin Name: API Posts Display
 * Description: Fetches and displays posts from JSONPlaceholder API with styled cards
 * Version: 1.0.0
 * Author: CodeFireSolution Test
 * Text Domain: api-posts-display
 */

if (!defined('ABSPATH')) {
    exit; // Exit if accessed directly
}

class API_Posts_Display {
    // Store our API endpoint
    private $api_url = 'https://jsonplaceholder.typicode.com/posts';
    
    // Define our transient key for caching
    private $transient_key = 'api_posts_cache';
    
    // Set cache duration to one hour
    private $cache_duration = 3600;

    public function __construct() {
        // Register our shortcode
        add_shortcode('display_api_posts', array($this, 'display_posts_shortcode'));
    }

    /**
     * Fetches posts from the API with caching and error handling
     */
    private function fetch_posts() {
        // Check for cached data first
        $cached_posts = get_transient($this->transient_key);
        
        if (false !== $cached_posts) {
            return $cached_posts;
        }

        // Make the API request
        $response = wp_remote_get(
            $this->api_url,
            array(
                'timeout' => 15,
                'headers' => array('Accept' => 'application/json')
            )
        );

        // Handle potential errors
        if (is_wp_error($response)) {
            return array('error' => $response->get_error_message());
        }

        // Process the response
        $body = wp_remote_retrieve_body($response);
        $posts = json_decode($body, true);

        if (empty($posts) || !is_array($posts)) {
            return array('error' => __('Invalid response from API', 'api-posts-display'));
        }

        // Limit to 5 posts
        $posts = array_slice($posts, 0, 5);
        
        // Cache the results
        set_transient($this->transient_key, $posts, $this->cache_duration);

        return $posts;
    }

    /**
     * Formats a single post into a card layout
     * The HTML structure matches the provided CSS classes for proper styling
     */
    private function format_post($post) {
        // Create a clean excerpt
        $excerpt = wp_trim_words($post['body'], 20, '...');
        
        // Calculate reading time
        $word_count = str_word_count(strip_tags($post['body']));
        $reading_time = max(1, ceil($word_count / 200));

        // Generate a random category for visual variety
        $categories = array('Technology', 'Lifestyle', 'Business', 'Design', 'Development');
        $category = $categories[array_rand($categories)];

        // Build the card HTML structure
        return sprintf(
            '
                
                    
                        %s
                        %d min read
                    
                    %s
                    %s
                    
                        
                            %s
                            →
                        
                    
                
            ',
            esc_html($category),
            esc_html($reading_time),
            esc_html($post['title']),
            esc_html($excerpt),
            esc_html__('Read More', 'api-posts-display')
        );
    }

    /**
     * Shortcode callback function that displays the posts in a grid layout
     */
    public function display_posts_shortcode($atts) {
        // Get posts from API
        $posts = $this->fetch_posts();

        // Handle any errors
        if (isset($posts['error'])) {
            return sprintf(
                '%s',
                esc_html($posts['error'])
            );
        }

        // Create the grid container and loop through posts
        $output = '';
        
        foreach ($posts as $post) {
            $output .= $this->format_post($post);
        }
        
        $output .= '';

        return $output;
    }
}

// Initialize the plugin
new API_Posts_Display();
```

## Core Components Explained

### Caching System

The plugin implements a sophisticated caching system using WordPress transients. This approach significantly improves performance by reducing unnecessary API calls. When a request is made, the plugin first checks for cached data using a unique transient key (`api_posts_cache`). The cache expires after one hour (3600 seconds), at which point a new API request will be made. This balance between fresh content and performance is particularly important when dealing with external APIs.

### API Integration

The integration with the JSONPlaceholder API is handled through WordPress's HTTP API (`wp_remote_get`). The implementation includes several important features:

1. A timeout setting of 15 seconds prevents hanging requests
2. Proper headers are set to request JSON data
3. Comprehensive error handling catches and reports any API issues
4. Response validation ensures the data is in the expected format
5. The results are limited to 5 posts to maintain a manageable display

### Post Formatting

The `format_post` method transforms raw API data into engaging card layouts. Each card includes:

1. A randomly assigned category for visual interest
2. An estimated reading time based on word count (assuming 200 words per minute)
3. A clean excerpt limited to 20 words
4. A consistent HTML structure with semantic markup
5. Proper escaping of all output data for security

### Display Implementation

The display logic is implemented through a WordPress shortcode (`[display_api_posts]`). The shortcode handler:

1. Retrieves posts (either from cache or API)
2. Handles any potential errors gracefully
3. Wraps posts in a grid container for layout
4. Applies consistent formatting to each post

## Security Considerations

The plugin implements several security measures:

1. Direct file access prevention through ABSPATH check
2. Data sanitization for API responses
3. Output escaping for all displayed content
4. Error handling to prevent exposure of sensitive information
5. Proper use of WordPress security functions

## Performance Features

Several performance optimizations are implemented:

1. Transient-based caching reduces API calls
2. Limited post count prevents overwhelming the display
3. Efficient HTML generation using sprintf
4. Minimal DOM structure for better rendering performance
5. Smart timeout handling for API requests

## Usage Instructions

### Basic Implementation

To display the API posts on any page or post, simply add the shortcode:

```
[display_api_posts]
```

### Styling Customization

The plugin uses semantic class names for easy styling. The main classes are:

1. `api-posts-container`: The grid container for all posts
2. `api-post-card`: Individual post card wrapper
3. `api-post-card-content`: Content area within each card
4. `api-post-meta`: Container for post metadata
5. `api-post-title`: Post title styling
6. `api-post-excerpt`: Post excerpt container
7. `api-post-footer`: Card footer with read more link


