*------------------------------------------- Botão case para selecionar em module pool

WHEN 'IMPORTAR'.
    PERFORM f_recebe_dados.

*------------------------------------------ Perform

FORM f_recebe_dados .

  DATA: v_tabela_arquivo TYPE filetable,
     v_return_code  TYPE i,
     vl_matnr     TYPE zsdt104_i-matnr,
     vl_prec1     TYPE waers,
     vl_prec2     TYPE waers,
     l_caminho    TYPE string.

  TYPES: BEGIN OF ty_excel,
      matnr TYPE zsdt104_i-matnr,
      prec1 LIKE vl_prec1,
      prec2 LIKE vl_prec2,
     END OF ty_excel.

  DATA: BEGIN OF t_upload OCCURS 0,
      campo(255),
     END OF t_upload.

  DATA: wa_zsdt104_i TYPE TABLE OF ty_excel WITH HEADER LINE,
     it_zsdt104_i TYPE TABLE OF ty_excel WITH HEADER LINE,
     wa_upload  LIKE t_upload.

  CASE sy-ucomm.

   WHEN 'IMPORTAR'.
    CALL METHOD cl_gui_frontend_services=>file_open_dialog
     EXPORTING
      file_filter       = 'CSV'
     CHANGING
      file_table       = v_tabela_arquivo
      rc           = v_return_code
     EXCEPTIONS
      file_open_dialog_failed = 1
      cntl_error       = 2
      error_no_gui      = 3
      not_supported_by_gui  = 4
      OTHERS         = 5.

    IF v_return_code = 1.
     READ TABLE v_tabela_arquivo INDEX 1 INTO l_caminho.
     
     FREE: t_upload.
     
     CALL FUNCTION 'GUI_UPLOAD'
      EXPORTING
       filename      = l_caminho
       filetype      = 'ASC'
       has_field_separator = ' '
      TABLES
       data_tab      = t_upload
      EXCEPTIONS
       file_open_error   = 1
       file_read_error   = 2
       no_batch      = 3
       OTHERS       = 17.
     
     IF sy-subrc = 0.
     
      FREE: it_zsdt104_i.

​      LOOP AT t_upload ASSIGNING FIELD-SYMBOL(<fs_upload>).

       CLEAR: wa_zsdt104_i, vl_matnr, vl_prec1, vl_prec2.
     
       SPLIT <fs_upload>-campo AT ';'
       INTO vl_matnr
         vl_prec1
         vl_prec2.
     
       wa_zsdt104_i-matnr = vl_matnr.
       wa_zsdt104_i-prec1 = vl_prec1.
       wa_zsdt104_i-prec2 = vl_prec2.
       APPEND wa_zsdt104_i TO it_zsdt104_i.
     
       IF vl_matnr IS NOT INITIAL.
     
        CALL FUNCTION 'CONVERSION_EXIT_ALPHA_INPUT'
         EXPORTING
          input = vl_matnr
         IMPORTING
          output = vl_matnr.
     
        READ TABLE gt_precos ASSIGNING FIELD-SYMBOL(<fs_precos>)
         WITH KEY matnr = vl_matnr.
     
        IF sy-subrc IS INITIAL.
     
         <fs_precos>-prec1 = vl_prec1.
         <fs_precos>-prec2 = vl_prec2.
     
        ENDIF.
     
       ENDIF.
     
      ENDLOOP.
     
      MESSAGE: 'Arquivo importado' TYPE 'S'.
      CALL SCREEN 9000.
     ENDIF.
    ENDIF.

  ENDCASE.
 ENDFORM.