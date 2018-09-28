# Post Loader
Renders posts via ajax.

## Usage

### Create post loader
    
    theme_create_post_loader( 'my_post_loader' );

### Display post loader

    theme_post_loader( 'my_post_loader' );
    
or by shortcode:

    [post-loader id="my_post_loader"]
    
### Enqueue plugin scripts

*Note: Scripts are automatically loaded when shortcode `[post-loader]` is used in the post content (on a single post or page).*

    function my_scripts()
    {
        theme_post_loader_enqueue_scripts();
    }

    add_action( 'wp_enqueue_scripts', 'my_scripts' );

## Actions

### theme_post_loader_inside/loader={loader_id}

Alter the loader elements.

    function my_post_loader_inside( $loader )
    {
        // Include form and content elements inside a grid.
        ?>
        
        <div class="row">
            <div class="col-lg-4">
                <?php $loader->form(); ?>
            </div>
            <div class="col">
                <?php $loader->content(); ?>
             </div>
        </div>
        
        <?php
    }
    
    add_action( 'theme_post_loader_inside/loader=my_post_loader', 'my_post_loader_inside' );

### theme_post_loader_form/loader={loader_id}

Alter the form.

    function my_post_loader_form( $loader )
    {
        // Create term filter

		$terms = get_terms( array
		(
			'taxonomy' => 'category'
		));

		?>

		<form class="post-loader-form" method="post">
		
			<?php 
			    // required: Output settings fields
			    $loader->settings_fields();
            ?>
			
			<?php if ( $terms ) : ?>
			<div class="term-filter">
				<?php foreach ( $terms as $term ) : ?>
				<label><input type="checkbox" class="autoload" name="terms[]" value="<?php echo esc_attr( $term->term_id ); ?>"> <?php echo esc_html( $term->name ); ?></label>
				<?php endforeach; ?>
			</div><!-- .term-filter -->
			<?php endif ?>

		</form><!-- .post-loader-form -->

		<?php
    }
    
    add_action( 'theme_post_loader_form/loader=my_post_loader', 'my_post_loader_form' );
    
### theme_post_loader_result/loader={loader_id}

Alter the result.

    function my_post_loader_result( $query, $loader )
    {
        // Check posts
		if ( $query->have_posts() ) 
		{
		    echo '<div class="row">';
		    
			// The Loop
    		while ( $query->have_posts() ) 
    		{
    			$query->the_post();
                
                echo '<div class="col-md-4">';
                 
    			// Include post template
    			get_template_part( 'content', get_post_type() );
    			
                echo '</div>'; // .col…
    		}
    		
    		 echo '</div>'; // .row
    		
    		// Pagination
    		$loader->pagination( $query );
    
    		// Reset post data
    		wp_reset_postdata();
		}
		
		else
		{
		    // Not found message
		    _e( 'No posts found' );
		}
    }
    
    add_action( 'theme_post_loader_result/loader=my_post_loader', 'my_post_loader_result', 10, 2 );

## Filters

### theme_post_loader_query_args/loader={loader_id}

Alter the WP Query arguments.

    function my_post_loader_query_args( $query_args, $loader )
    {
        // Apply term filter
        
        $terms = isset( $_POST['terms'] ) && is_array( $_POST['terms'] ) ? $_POST['terms'] : array();
        
        if ( $terms ) 
		{
			$query_args['tax_query'][] = array
			(
				'taxonomy' => 'category',
				'field'    => 'term_id',
				'terms'    => array_map( 'intval', $terms ),
				'operator' => 'IN',
			);
		}
        
        // Return
        return $query_args;
    }
    
    add_filter( 'theme_post_loader_query_args/loader=my_post_loader', 'my_post_loader_query_args', 10, 2 );
    
## OOP

For an object oriented approach see example file `includes/class-theme-sample-post-loader.php`.