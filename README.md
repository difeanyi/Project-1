# Project-1
# project_1.s
# MIPS assembly program to process a 10-character input string
# and compute G - H based on a custom base-N system.
#
# Assumptions:
# - N is hardcoded based on the Howard ID.
# - The program reads exactly 10 characters from input.
# - Valid characters are converted to base-N values.
# - Computes (G - H) and prints the result, or "N/A" if no valid digits exist.

.data
    input: .space 11      # Buffer to store the input string (10 chars + null terminator)
    output_na: .asciiz "N/A\n"
    output_newline: .asciiz "\n"

.text
    .globl main

main:
    # Read 10 characters from user input
    li $v0, 8          # syscall for string input
    la $a0, input      # Load address of input buffer
    li $a1, 11         # Read up to 10 characters + null terminator
    syscall

    # Hardcoded N based on (Howard ID % 11) + 26
    li $t0, 27         # N = 27 
    li $t1, 7         # M = N - 10 = 7
    
    # Define valid ranges
    li $t2, '0'        # ASCII '0'
    li $t3, '9'        # ASCII '9'
    li $t4, 'a'        # ASCII 'a'
    add $t5, $t4, $t1  # ASCII 'β' (M-th lowercase letter)
    li $t6, 'A'        # ASCII 'A'
    add $t7, $t6, $t1  # ASCII 'Δ' (M-th uppercase letter)
    
    # Initialize counters
    li $s0, 0          # G (sum of first 5 valid digits)
    li $s1, 0          # H (sum of last 5 valid digits)
    li $s2, 0          # Valid character count

    la $a0, input      # Load input buffer address
    li $t8, 0          # Index counter

process_chars:
    lb $t9, 0($a0)     # Load character
    beqz $t9, check_output  # If null terminator, end loop
    
    # Check if valid digit ('0' to '9')
    blt $t9, $t2, next_char
    ble $t9, $t3, convert_digit
    
    # Check if valid lowercase ('a' to β)
    blt $t9, $t4, next_char
    ble $t9, $t5, convert_letter
    
    # Check if valid uppercase ('A' to Δ)
    blt $t9, $t6, next_char
    ble $t9, $t7, convert_letter
    
    j next_char  # Skip invalid characters
    
convert_digit:
    sub $t9, $t9, $t2   # Convert '0'-'9' to decimal (0-9)
    j store_value
    
convert_letter:
    or $t9, $t9, 0      # Already in valid range, just convert
    sub $t9, $t9, $t4   # Convert to base-N equivalent
    addi $t9, $t9, 10   # Adjust for base-N mapping ('a' = 10, etc.)
    
store_value:
    blt $t8, 5, store_G # First 5 go to G, rest go to H
    add $s1, $s1, $t9   # Add to H
    j next_count
    
store_G:
    add $s0, $s0, $t9   # Add to G
    
next_count:
    addi $s2, $s2, 1    # Increment valid count
    
next_char:
    addi $a0, $a0, 1    # Move to next char in input
    addi $t8, $t8, 1    # Increment index
    j process_chars
    
check_output:
    beqz $s2, print_na  # If no valid characters, print "N/A"
    sub $s3, $s0, $s1   # Compute G - H
    li $v0, 1           # Print integer syscall
    move $a0, $s3       # Load result
    syscall
    j exit

print_na:
    li $v0, 4           # Print string syscall
    la $a0, output_na   # Load "N/A" message
    syscall

exit:
    li $v0, 10          # Exit syscall
    syscall
