<?php
/**
 * Plugin Name: Imágenes No Usadas
 * Description: Detecta imágenes no utilizadas en la biblioteca de medios de WordPress y permite eliminarlas.
 * Version: 1.2
 * Author: Tu Nombre
 */

if (!defined('ABSPATH')) {
    exit; // Evitar accesos directos
}

function detectar_imagenes_no_usadas() {
    global $wpdb;

    // Obtener todas las imágenes de la biblioteca de medios
    $imagenes = $wpdb->get_results("SELECT ID, guid FROM {$wpdb->posts} WHERE post_type = 'attachment' AND post_mime_type LIKE 'image/%'");
    $imagenes_no_usadas = [];

    foreach ($imagenes as $imagen) {
        // Verificar si la imagen está en uso en el contenido de algún post, página o custom post type
        $usada = $wpdb->get_var($wpdb->prepare("SELECT COUNT(*) FROM {$wpdb->posts} WHERE post_content LIKE %s", '%' . $imagen->guid . '%'));

        if (!$usada) {
            $imagenes_no_usadas[] = [
                'id' => $imagen->ID,
                'url' => $imagen->guid,
                'thumbnail' => wp_get_attachment_image_src($imagen->ID, 'thumbnail')[0] // Miniatura de la imagen
            ];
        }
    }

    // Mostrar resultados en formato tabla
    echo '<div class="wrap"><h2>Imágenes No Usadas</h2>';
    if (!empty($imagenes_no_usadas)) {
        echo '<table class="wp-list-table widefat fixed striped posts">';
        echo '<thead><tr><th>Vista Previa</th><th>URL</th><th>Acciones</th></tr></thead>';
        echo '<tbody>';

        foreach ($imagenes_no_usadas as $img) {
            echo '<tr>';
            echo '<td><img src="' . esc_url($img['thumbnail']) . '" alt="Vista previa" style="max-width: 100px; height: auto;" /></td>';
            echo '<td><a href="' . esc_url($img['url']) . '" target="_blank">' . esc_html($img['url']) . '</a></td>';
            echo '<td>';
            echo '<form method="post" style="display:inline;">
                    <input type="hidden" name="imagen_id" value="' . esc_attr($img['id']) . '">
                    <input type="submit" name="borrar_imagen" value="Eliminar" onclick="return confirm(\'¿Estás seguro de que deseas eliminar esta imagen?\');" class="button button-danger">
                  </form>';
            echo '</td>';
            echo '</tr>';
        }

        echo '</tbody>';
        echo '</table>';
    } else {
        echo '<p>No se encontraron imágenes no utilizadas.</p>';
    }
    echo '</div>';

    // Eliminar imagen si el formulario de eliminación fue enviado
    if (isset($_POST['borrar_imagen'])) {
        $imagen_id = intval($_POST['imagen_id']);
        wp_delete_attachment($imagen_id, true); // Elimina la imagen permanentemente
        echo '<div class="updated"><p>Imagen eliminada correctamente.</p></div>';
    }
}

function registrar_pagina_imagenes_no_usadas() {
    add_menu_page(
        'Imágenes No Usadas',
        'Imágenes No Usadas',
        'manage_options',
        'imagenes-no-usadas',
        'detectar_imagenes_no_usadas',
        'dashicons-hidden',
        20
    );
}
add_action('admin_menu', 'registrar_pagina_imagenes_no_usadas');