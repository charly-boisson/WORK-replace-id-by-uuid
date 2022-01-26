# WORK-replace-id-by-uuid

## Objectif :

Remplacer les id par des uuid pour exporter des données d'un environnement à un autre

## Informations :
 - Feature principale : refacto-uuid
 - Step 1 BDD (charly) : refacto-uuid-step1
 - Step 2 Back-end (Aurélien) : refacto-uuid-step2
 - Step 3 Front-end (Anthony) : refacto-uuid-step3
 -  Support : Kévin / Brice / Léo


## Step 1 : Détail du processus de migration de la Base de données

    // ---------------------------------------------------------// 
    CREATE  COLUMNS TMP FOR TRANSFERT  
    // ---------------------------------------------------------  
            
        DB::statement('ALTER TABLE p4it_atelier_grid_items ADD COLUMN report_uuid uuid NOT NULL DEFAULT uuid_generate_v1();');
        DB::statement('ALTER TABLE p4it_atelier_grid_items ADD COLUMN parent_uuid uuid;'); 
        DB::statement('ALTER TABLE p4it_atelier_grid_items ADD COLUMN statement_uuid uuid;');
        DB::statement('ALTER TABLE p4it_atelier_grid_items ADD COLUMN original_uuid uuid;');

    // ---------------------------------------------------------// 
    TRANSFERT ID => UUID  
    // ---------------------------------------------------------
    
    Schema::_table_('p4it_atelier_grid_items', function (Blueprint $table) {
      
    DB::_statement_('UPDATE p4it_atelier_grid_items set report_uuid = p4it_atelier_reports.uuid from p4it_atelier_reports where p4it_atelier_reports.id = p4it_atelier_grid_items.report_id;');
    
    DB::_statement_('UPDATE p4it_atelier_grid_items as grid_item1 set parent_uuid = grid_item2.uuid from p4it_atelier_grid_items as grid_item2 where grid_item1.parent_id = grid_item2.id;');

    DB::_statement_('UPDATE p4it_atelier_grid_items set statement_uuid = p4it_atelier_statements.uuid from p4it_atelier_statements where p4it_atelier_statements.id = p4it_atelier_grid_items.statement_id;');

    DB::_statement_('UPDATE p4it_atelier_grid_items as grid_item1 set original_uuid = grid_item2.uuid from p4it_atelier_grid_items as grid_item2 where grid_item1.original_id = grid_item2.id;');
	
	// ETC...
    // GRID ITEMS (options) => VOIR MIGRATION
    
    // ---------------------------------------------------------
    // DELETE All CONSTRAINTS
    // ---------------------------------------------------------
    
    // GRID ITEMS
    DB::statement('ALTER TABLE p4it_atelier_grid_items DROP CONSTRAINT IF EXISTS p4it_atelier_grid_items_report_id_foreign;');
    
    DB::statement('ALTER TABLE p4it_atelier_grid_items DROP CONSTRAINT IF EXISTS p4it_atelier_grid_items_parent_id_foreign;');
    
    DB::statement('ALTER TABLE p4it_atelier_grid_items DROP CONSTRAINT IF EXISTS p4it_atelier_grid_items_pkey;');

    // ---------------------------------------------------------
    // DROP AND RENAME COLUMNS
    // ---------------------------------------------------------
    
    // GRID ITEMS
    Schema::table('p4it_atelier_grid_items', function (Blueprint $table) {
    
      // GRID ITEMS (Relationship report)
      $table->dropColumn('report_id');
      $table->renameColumn('report_uuid', 'report_id');
    
      // GRID ITEMS (parent_id grid item)
      $table->dropColumn('parent_id');
      $table->renameColumn('parent_uuid', 'parent_id');
    
      // GRID ITEMS (parent_id grid item)
      $table->dropColumn('statement_id');
      $table->renameColumn('statement_uuid', 'statement_id');
    
      // GRID ITEMS (original_id grid item)
      $table->dropColumn('original_id');
      $table->renameColumn('original_uuid', 'original_id');
    
      // ID - UUID
      $table->dropColumn('id');
      $table->renameColumn('uuid', 'id');
    });
