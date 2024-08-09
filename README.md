# Workday-to-ECC-HR-Mini-Master
Workday Integration with HR Mini master in ECC using CPI, HRMD_A09 IDOC

## Development 1
Change pointer data in table bdcp2 not generated for Inbound IDOC.Reference 1718859 - HRALE: New BAdI for application change pointer generation

## Solution 1
Implemented BAdI Definition Name      HRALE00APPL_CHG_PTS to write change pointers for message type CTE_HCM_EMPLOYEE(Concur)
```
Method: IF_EX_HRALE00APPL_CHG_PTS~GEN_CHANGE_POINTERS
  DATA:
    lt_change_pointers TYPE hrobjinfty_tab.

* only the PA-infotype data has to be processed
  LOOP AT t_hrplogi INTO DATA(ls_hrplogi)
    WHERE otype = 'P'.

    CASE ls_hrplogi-opera.
      WHEN 'D'.
      WHEN 'U' OR 'I'.
*       hrplogi-opera = 'U' means: in the given time interval hrobjinfty-begda -
*       hrobjinfty-endda all existing hrobjinfty-infty/subty records are deleted
*       and the given hrobjsdata-records are inserted
*       => generic change pointers with operation 'U' (update) are written

        lt_change_pointers = VALUE hrobjinfty_tab(
          BASE lt_change_pointers
            FOR  ls_hrobjinfty IN t_hrobjinfty
              WHERE ( otype = ls_hrplogi-otype
                  AND objid = ls_hrplogi-objid )
               ( plvar = ls_hrobjinfty-plvar
                 otype = ls_hrobjinfty-otype
                 objid = ls_hrobjinfty-objid
                 infty = ls_hrobjinfty-infty
                 subty = ls_hrobjinfty-subty
                 begda = ls_hrobjinfty-begda
                 endda = ls_hrobjinfty-endda
                 segnum = ls_hrobjinfty-segnum ) ).
    ENDCASE.
  ENDLOOP.
  DATA: badi_change_ptrs_i TYPE REF TO if_ex_hrale00change_ptrs_i.
  cl_exithandler=>get_instance( CHANGING instance = badi_change_ptrs_i ).

  badi_change_ptrs_i->set_last_change(
    CHANGING t_changed_objects = lt_change_pointers[] ).

  badi_change_ptrs_i->process_change_pointers(
    CHANGING
      t_changed_objects = lt_change_pointers[] ).
“For other message types following FM would help.
*  CALL FUNCTION 'CHANGE_POINTERS_CREATE_DIRECT'
*    EXPORTING
*      message_type          = message_type
*    TABLES
*      t_cp_data             = t_cp_data
*    EXCEPTIONS
*      number_range_problems = 4
*      OTHERS                = 8.
*  CASE sy-subrc.
*    WHEN 0.
*    WHEN 4.
*      RAISE number_range_problems.
*    WHEN 8.
*    WHEN OTHERS.
*  ENDCASE.
```


## Development 2
## Development 3
## Development 4
## Development 5
## Development 6
## Development 7
