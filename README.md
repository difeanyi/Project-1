        .data
# Do not print any extra messages or prompts.
buffer: .space 11                # Space for 10 characters plus null terminator.
naStr:  .asciiz "N/A"            # String to print if no valid digits are found.

        .text
        .globl main

# For Howard ID: 04004740, we have N = 29 and M = 19.
# Valid digits:
#  - '0' to '9' represent 0-9.
#  - 'a' to 's' (ASCII 97 to 115) represent 10 to 28.
#  - 'A' to 'S' (ASCII 65 to 83) represent 10 to 28.

main:
    # Read exactly 10-character input from the user.
    li      $v0, 8             # Syscall: read_string.
    la      $a0, buffer        # Address to store the input.
    li      $a1, 11            # Read up to 10 characters plus the null terminator.
    syscall

    # Initialize index counter and sums:
    li      $t0, 0             # Character index (0 to 9).
    li      $s0, 0             # Sum of first half (G).
    li      $s1, 0             # Sum of second half (H).
    li      $s2, 0             # Counter for valid digits.

loop:
    li      $t2, 10            # Total number of characters to process.
    bge     $t0, $t2, done_loop    # Exit loop if index >= 10.

    # Load current character from buffer.
    la      $t3, buffer        
    add     $t3, $t3, $t0      # Compute address: buffer + index.
    lb      $t4, 0($t3)        # Load the character into $t4.

    # ----- Check if the character is a digit ('0'-'9') -----
    li      $t5, 48            # ASCII for '0'.
    li      $t6, 57            # ASCII for '9'.
    blt     $t4, $t5, check_lower    # If char < '0', check letters.
    bgt     $t4, $t6, check_lower    # If char > '9', check letters.

    # Character is a valid digit. Compute its value: char - '0'.
    sub     $t7, $t4, $t5      # $t7 now holds the digit value (0–9).
    j       process_digit

check_lower:
    # ----- Check if character is a valid lowercase letter ('a'-'s') -----
    li      $t5, 97            # ASCII for 'a'.
    li      $t6, 115           # ASCII for 's' (since M = 19, valid letters: a-s).
    blt     $t4, $t5, check_upper  # If char < 'a', check uppercase.
    bgt     $t4, $t6, check_upper  # If char > 's', not valid; check uppercase.

    # Valid lowercase letter. Compute value: (char - 'a' + 10).
    sub     $t7, $t4, $t5      # $t7 = (char - 'a').
    addi    $t7, $t7, 10       # Adjust value so that 'a' maps to 10.
    j       process_digit

check_upper:
    # ----- Check if character is a valid uppercase letter ('A'-'S') -----
    li      $t5, 65            # ASCII for 'A'.
    li      $t6, 83            # ASCII for 'S' (since M = 19, valid letters: A-S).
    blt     $t4, $t5, skip_char    # If char < 'A', skip this character.
    bgt     $t4, $t6, skip_char    # If char > 'S', skip this character.

    # Valid uppercase letter. Compute its value: (char - 'A' + 10).
    sub     $t7, $t4, $t5      # $t7 = (char - 'A').
    addi    $t7, $t7, 10       # Adjust so that 'A' maps to 10.
    # Continue processing as a valid digit.

process_digit:
    # Increment the valid digit counter.
    addi    $s2, $s2, 1

    # Decide which half of the input this character belongs to.
    li      $t8, 5             # First half: positions 0-4.
    blt     $t0, $t8, add_to_first  # If index < 5, add value to G.
    
    # Otherwise, add the value to the second half sum H.
    add     $s1, $s1, $t7
    j       skip_char

add_to_first:
    add     $s0, $s0, $t7      # Add valid digit value to G.

skip_char:
    # Increment the index and continue the loop.
    addi    $t0, $t0, 1        
    j       loop

done_loop:
    # If no valid digits were found, print "N/A".
    beq     $s2, $zero, no_valid

    # Otherwise, compute result = G - H.
    sub     $t9, $s0, $s1

    # Print the result as a decimal integer.
    li      $v0, 1             # Syscall: print integer.
    move    $a0, $t9           
    syscall
    j       exit_program

no_valid:
    li      $v0, 4             # Syscall: print string.
    la      $a0, naStr         
    syscall

exit_program:
    li      $v0, 10            # Syscall: exit.
    syscall

   
  

   
    

   
