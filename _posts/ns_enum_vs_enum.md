ns_enum vs enum - reversing

```objc

#import <Foundation/Foundation.h>

typedef enum : NSUInteger {
    EnumTypeOption0,
    EnumTypeOption1,
} EnumType;

typedef NS_ENUM(NSUInteger, NSEnumType) {
    NSEnumTypeOption0,
    NSEnumTypeOption1,
};

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        NSLog(@"Hello, World!");
    }
    return 0;
}

```

``` asm
# Assembly output for main.m
# Generated at 1:05:48 AM on Monday, May 1, 2017
# Using Release configuration, x86_64 architecture for ns_enum_vs_enum target of ns_enum_vs_enum project

	.section	__TEXT,__text,regular,pure_instructions
	.macosx_version_min 10, 12
	.file	1 "/Users/yurylapitsky/xcode_projects/ns_enum_vs_enum" "/Users/yurylapitsky/xcode_projects/ns_enum_vs_enum/ns_enum_vs_enum/main.m"
	.private_extern	_main
	.globl	_main
_main:                                  ## @main
Lfunc_begin0:
	.loc	1 21 0                  ## /Users/yurylapitsky/xcode_projects/ns_enum_vs_enum/ns_enum_vs_enum/main.m:21:0
	.cfi_startproc
## BB#0:
	pushq	%rbp
Ltmp0:
	.cfi_def_cfa_offset 16
Ltmp1:
	.cfi_offset %rbp, -16
	movq	%rsp, %rbp
Ltmp2:
	.cfi_def_cfa_register %rbp
Ltmp3:
	.loc	1 22 22 prologue_end    ## /Users/yurylapitsky/xcode_projects/ns_enum_vs_enum/ns_enum_vs_enum/main.m:22:22
	pushq	%rbx
	pushq	%rax
Ltmp4:
	.cfi_offset %rbx, -24
	##DEBUG_VALUE: main:argc <- %EDI
	##DEBUG_VALUE: main:argv <- %RSI
	callq	_objc_autoreleasePoolPush
Ltmp5:
	movq	%rax, %rbx
Ltmp6:
	.loc	1 24 9                  ## /Users/yurylapitsky/xcode_projects/ns_enum_vs_enum/ns_enum_vs_enum/main.m:24:9
	leaq	L__unnamed_cfstring_(%rip), %rdi
	xorl	%eax, %eax
	callq	_NSLog
	.loc	1 25 5                  ## /Users/yurylapitsky/xcode_projects/ns_enum_vs_enum/ns_enum_vs_enum/main.m:25:5
	movq	%rbx, %rdi
	callq	_objc_autoreleasePoolPop
Ltmp7:
	.loc	1 26 5                  ## /Users/yurylapitsky/xcode_projects/ns_enum_vs_enum/ns_enum_vs_enum/main.m:26:5
	xorl	%eax, %eax
	addq	$8, %rsp
	popq	%rbx
	popq	%rbp
	retq
Ltmp8:
Lfunc_end0:
	.cfi_endproc

	.section	__TEXT,__cstring,cstring_literals
L_.str:                                 ## @.str
	.asciz	"Hello, World!"

	.section	__DATA,__cfstring
	.p2align	3               ## @_unnamed_cfstring_
L__unnamed_cfstring_:
	.quad	___CFConstantStringClassReference
	.long	1992                    ## 0x7c8
	.space	4
	.quad	L_.str
	.quad	13                      ## 0xd

	.linker_option "-framework", "Foundation"
	.linker_option "-framework", "ApplicationServices"
	.linker_option "-framework", "CoreText"
	.linker_option "-framework", "CoreServices"
	.linker_option "-framework", "DiskArbitration"
	.linker_option "-framework", "CFNetwork"
	.linker_option "-framework", "Security"
	.linker_option "-framework", "ImageIO"
	.linker_option "-framework", "CoreGraphics"
	.linker_option "-framework", "IOKit"
	.linker_option "-framework", "CoreFoundation"
	.section	__DATA,__objc_imageinfo,regular,no_dead_strip
L_OBJC_IMAGE_INFO:
	.long	0
	.long	64

	.section	__DWARF,__debug_str,regular,debug
Linfo_string:
	.asciz	"Apple LLVM version 8.1.0 (clang-802.0.41)" ## string offset=0
	.asciz	"/Users/yurylapitsky/xcode_projects/ns_enum_vs_enum/ns_enum_vs_enum/main.m" ## string offset=42
	.asciz	"/Users/yurylapitsky/xcode_projects/ns_enum_vs_enum" ## string offset=116
	.asciz	"Foundation"            ## string offset=167
	.asciz	"\"-DNS_BLOCK_ASSERTIONS=1\" \"-DOBJC_OLD_DISPATCH_PROTOTYPES=0\"" ## string offset=178
	.asciz	"/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk/System/Library/Frameworks/Foundation.framework" ## string offset=239
	.asciz	"/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk" ## string offset=386
	.asciz	"/Users/yurylapitsky/Library/Developer/Xcode/DerivedData/ModuleCache/137YIV15VC76S/Foundation-2FJBXN8U6QRTS.pcm" ## string offset=486
	.asciz	"main"                  ## string offset=597
	.asciz	"int"                   ## string offset=602
	.asciz	"argc"                  ## string offset=606
	.asciz	"argv"                  ## string offset=611
	.asciz	"char"                  ## string offset=616
	.section	__DWARF,__debug_loc,regular,debug
Lsection_debug_loc:
Ldebug_loc0:
Lset0 = Lfunc_begin0-Lfunc_begin0
	.quad	Lset0
Lset1 = Ltmp5-Lfunc_begin0
	.quad	Lset1
	.short	3                       ## Loc expr size
	.byte	85                      ## super-register DW_OP_reg5
	.byte	147                     ## DW_OP_piece
	.byte	4                       ## 4
	.quad	0
	.quad	0
Ldebug_loc1:
Lset2 = Lfunc_begin0-Lfunc_begin0
	.quad	Lset2
Lset3 = Ltmp5-Lfunc_begin0
	.quad	Lset3
	.short	1                       ## Loc expr size
	.byte	84                      ## DW_OP_reg4
	.quad	0
	.quad	0
	.section	__DWARF,__debug_abbrev,regular,debug
Lsection_abbrev:
	.byte	1                       ## Abbreviation Code
	.byte	17                      ## DW_TAG_compile_unit
	.byte	1                       ## DW_CHILDREN_yes
	.byte	37                      ## DW_AT_producer
	.byte	14                      ## DW_FORM_strp
	.byte	19                      ## DW_AT_language
	.byte	5                       ## DW_FORM_data2
	.byte	3                       ## DW_AT_name
	.byte	14                      ## DW_FORM_strp
	.byte	16                      ## DW_AT_stmt_list
	.byte	23                      ## DW_FORM_sec_offset
	.byte	27                      ## DW_AT_comp_dir
	.byte	14                      ## DW_FORM_strp
	.ascii	"\341\177"              ## DW_AT_APPLE_optimized
	.byte	25                      ## DW_FORM_flag_present
	.ascii	"\345\177"              ## DW_AT_APPLE_major_runtime_vers
	.byte	11                      ## DW_FORM_data1
	.byte	17                      ## DW_AT_low_pc
	.byte	1                       ## DW_FORM_addr
	.byte	18                      ## DW_AT_high_pc
	.byte	6                       ## DW_FORM_data4
	.byte	0                       ## EOM(1)
	.byte	0                       ## EOM(2)
	.byte	2                       ## Abbreviation Code
	.byte	30                      ## DW_TAG_module
	.byte	0                       ## DW_CHILDREN_no
	.byte	3                       ## DW_AT_name
	.byte	14                      ## DW_FORM_strp
	.ascii	"\201|"                 ## DW_AT_LLVM_config_macros
	.byte	14                      ## DW_FORM_strp
	.ascii	"\200|"                 ## DW_AT_LLVM_include_path
	.byte	14                      ## DW_FORM_strp
	.ascii	"\202|"                 ## DW_AT_LLVM_isysroot
	.byte	14                      ## DW_FORM_strp
	.byte	0                       ## EOM(1)
	.byte	0                       ## EOM(2)
	.byte	3                       ## Abbreviation Code
	.byte	8                       ## DW_TAG_imported_declaration
	.byte	0                       ## DW_CHILDREN_no
	.byte	58                      ## DW_AT_decl_file
	.byte	11                      ## DW_FORM_data1
	.byte	59                      ## DW_AT_decl_line
	.byte	11                      ## DW_FORM_data1
	.byte	24                      ## DW_AT_import
	.byte	19                      ## DW_FORM_ref4
	.byte	0                       ## EOM(1)
	.byte	0                       ## EOM(2)
	.byte	4                       ## Abbreviation Code
	.byte	46                      ## DW_TAG_subprogram
	.byte	1                       ## DW_CHILDREN_yes
	.byte	17                      ## DW_AT_low_pc
	.byte	1                       ## DW_FORM_addr
	.byte	18                      ## DW_AT_high_pc
	.byte	6                       ## DW_FORM_data4
	.byte	64                      ## DW_AT_frame_base
	.byte	24                      ## DW_FORM_exprloc
	.byte	3                       ## DW_AT_name
	.byte	14                      ## DW_FORM_strp
	.byte	58                      ## DW_AT_decl_file
	.byte	11                      ## DW_FORM_data1
	.byte	59                      ## DW_AT_decl_line
	.byte	11                      ## DW_FORM_data1
	.byte	39                      ## DW_AT_prototyped
	.byte	25                      ## DW_FORM_flag_present
	.byte	73                      ## DW_AT_type
	.byte	19                      ## DW_FORM_ref4
	.byte	63                      ## DW_AT_external
	.byte	25                      ## DW_FORM_flag_present
	.ascii	"\341\177"              ## DW_AT_APPLE_optimized
	.byte	25                      ## DW_FORM_flag_present
	.byte	0                       ## EOM(1)
	.byte	0                       ## EOM(2)
	.byte	5                       ## Abbreviation Code
	.byte	5                       ## DW_TAG_formal_parameter
	.byte	0                       ## DW_CHILDREN_no
	.byte	2                       ## DW_AT_location
	.byte	23                      ## DW_FORM_sec_offset
	.byte	3                       ## DW_AT_name
	.byte	14                      ## DW_FORM_strp
	.byte	58                      ## DW_AT_decl_file
	.byte	11                      ## DW_FORM_data1
	.byte	59                      ## DW_AT_decl_line
	.byte	11                      ## DW_FORM_data1
	.byte	73                      ## DW_AT_type
	.byte	19                      ## DW_FORM_ref4
	.byte	0                       ## EOM(1)
	.byte	0                       ## EOM(2)
	.byte	6                       ## Abbreviation Code
	.byte	36                      ## DW_TAG_base_type
	.byte	0                       ## DW_CHILDREN_no
	.byte	3                       ## DW_AT_name
	.byte	14                      ## DW_FORM_strp
	.byte	62                      ## DW_AT_encoding
	.byte	11                      ## DW_FORM_data1
	.byte	11                      ## DW_AT_byte_size
	.byte	11                      ## DW_FORM_data1
	.byte	0                       ## EOM(1)
	.byte	0                       ## EOM(2)
	.byte	7                       ## Abbreviation Code
	.byte	15                      ## DW_TAG_pointer_type
	.byte	0                       ## DW_CHILDREN_no
	.byte	73                      ## DW_AT_type
	.byte	19                      ## DW_FORM_ref4
	.byte	0                       ## EOM(1)
	.byte	0                       ## EOM(2)
	.byte	8                       ## Abbreviation Code
	.byte	38                      ## DW_TAG_const_type
	.byte	0                       ## DW_CHILDREN_no
	.byte	73                      ## DW_AT_type
	.byte	19                      ## DW_FORM_ref4
	.byte	0                       ## EOM(1)
	.byte	0                       ## EOM(2)
	.byte	9                       ## Abbreviation Code
	.byte	17                      ## DW_TAG_compile_unit
	.byte	0                       ## DW_CHILDREN_no
	.byte	37                      ## DW_AT_producer
	.byte	14                      ## DW_FORM_strp
	.byte	19                      ## DW_AT_language
	.byte	5                       ## DW_FORM_data2
	.byte	3                       ## DW_AT_name
	.byte	14                      ## DW_FORM_strp
	.byte	16                      ## DW_AT_stmt_list
	.byte	23                      ## DW_FORM_sec_offset
	.byte	27                      ## DW_AT_comp_dir
	.byte	14                      ## DW_FORM_strp
	.ascii	"\341\177"              ## DW_AT_APPLE_optimized
	.byte	25                      ## DW_FORM_flag_present
	.ascii	"\261B"                 ## DW_AT_GNU_dwo_id
	.byte	7                       ## DW_FORM_data8
	.ascii	"\260B"                 ## DW_AT_GNU_dwo_name
	.byte	14                      ## DW_FORM_strp
	.byte	0                       ## EOM(1)
	.byte	0                       ## EOM(2)
	.byte	0                       ## EOM(3)
	.section	__DWARF,__debug_info,regular,debug
Lsection_info:
Lcu_begin0:
	.long	149                     ## Length of Unit
	.short	4                       ## DWARF version number
Lset4 = Lsection_abbrev-Lsection_abbrev ## Offset Into Abbrev. Section
	.long	Lset4
	.byte	8                       ## Address Size (in bytes)
	.byte	1                       ## Abbrev [1] 0xb:0x8e DW_TAG_compile_unit
	.long	0                       ## DW_AT_producer
	.short	16                      ## DW_AT_language
	.long	42                      ## DW_AT_name
Lset5 = Lline_table_start0-Lsection_line ## DW_AT_stmt_list
	.long	Lset5
	.long	116                     ## DW_AT_comp_dir
                                        ## DW_AT_APPLE_optimized
	.byte	2                       ## DW_AT_APPLE_major_runtime_vers
	.quad	Lfunc_begin0            ## DW_AT_low_pc
Lset6 = Lfunc_end0-Lfunc_begin0         ## DW_AT_high_pc
	.long	Lset6
	.byte	2                       ## Abbrev [2] 0x2b:0x11 DW_TAG_module
	.long	167                     ## DW_AT_name
	.long	178                     ## DW_AT_LLVM_config_macros
	.long	239                     ## DW_AT_LLVM_include_path
	.long	386                     ## DW_AT_LLVM_isysroot
	.byte	3                       ## Abbrev [3] 0x3c:0x7 DW_TAG_imported_declaration
	.byte	1                       ## DW_AT_decl_file
	.byte	9                       ## DW_AT_decl_line
	.long	43                      ## DW_AT_import
	.byte	4                       ## Abbrev [4] 0x43:0x38 DW_TAG_subprogram
	.quad	Lfunc_begin0            ## DW_AT_low_pc
Lset7 = Lfunc_end0-Lfunc_begin0         ## DW_AT_high_pc
	.long	Lset7
	.byte	1                       ## DW_AT_frame_base
	.byte	86
	.long	597                     ## DW_AT_name
	.byte	1                       ## DW_AT_decl_file
	.byte	21                      ## DW_AT_decl_line
                                        ## DW_AT_prototyped
	.long	123                     ## DW_AT_type
                                        ## DW_AT_external
                                        ## DW_AT_APPLE_optimized
	.byte	5                       ## Abbrev [5] 0x5c:0xf DW_TAG_formal_parameter
Lset8 = Ldebug_loc0-Lsection_debug_loc  ## DW_AT_location
	.long	Lset8
	.long	606                     ## DW_AT_name
	.byte	1                       ## DW_AT_decl_file
	.byte	21                      ## DW_AT_decl_line
	.long	123                     ## DW_AT_type
	.byte	5                       ## Abbrev [5] 0x6b:0xf DW_TAG_formal_parameter
Lset9 = Ldebug_loc1-Lsection_debug_loc  ## DW_AT_location
	.long	Lset9
	.long	611                     ## DW_AT_name
	.byte	1                       ## DW_AT_decl_file
	.byte	21                      ## DW_AT_decl_line
	.long	130                     ## DW_AT_type
	.byte	0                       ## End Of Children Mark
	.byte	6                       ## Abbrev [6] 0x7b:0x7 DW_TAG_base_type
	.long	602                     ## DW_AT_name
	.byte	5                       ## DW_AT_encoding
	.byte	4                       ## DW_AT_byte_size
	.byte	7                       ## Abbrev [7] 0x82:0x5 DW_TAG_pointer_type
	.long	135                     ## DW_AT_type
	.byte	7                       ## Abbrev [7] 0x87:0x5 DW_TAG_pointer_type
	.long	140                     ## DW_AT_type
	.byte	8                       ## Abbrev [8] 0x8c:0x5 DW_TAG_const_type
	.long	145                     ## DW_AT_type
	.byte	6                       ## Abbrev [6] 0x91:0x7 DW_TAG_base_type
	.long	616                     ## DW_AT_name
	.byte	6                       ## DW_AT_encoding
	.byte	1                       ## DW_AT_byte_size
	.byte	0                       ## End Of Children Mark
Lcu_begin1:
	.long	38                      ## Length of Unit
	.short	4                       ## DWARF version number
Lset10 = Lsection_abbrev-Lsection_abbrev ## Offset Into Abbrev. Section
	.long	Lset10
	.byte	8                       ## Address Size (in bytes)
	.byte	9                       ## Abbrev [9] 0xb:0x1f DW_TAG_compile_unit
	.long	0                       ## DW_AT_producer
	.short	16                      ## DW_AT_language
	.long	167                     ## DW_AT_name
Lset11 = Lline_table_start0-Lsection_line ## DW_AT_stmt_list
	.long	Lset11
	.long	239                     ## DW_AT_comp_dir
                                        ## DW_AT_APPLE_optimized
	.quad	-6959349492065110333    ## DW_AT_GNU_dwo_id
	.long	486                     ## DW_AT_GNU_dwo_name
	.section	__DWARF,__debug_ranges,regular,debug
Ldebug_range:
	.section	__DWARF,__debug_macinfo,regular,debug
Ldebug_macinfo:
Lcu_macro_begin0:
Lcu_macro_begin1:
	.byte	0                       ## End Of Macro List Mark
	.section	__DWARF,__apple_names,regular,debug
Lnames_begin:
	.long	1212240712              ## Header Magic
	.short	1                       ## Header Version
	.short	0                       ## Header Hash Function
	.long	1                       ## Header Bucket Count
	.long	1                       ## Header Hash Count
	.long	12                      ## Header Data Length
	.long	0                       ## HeaderData Die Offset Base
	.long	1                       ## HeaderData Atom Count
	.short	1                       ## DW_ATOM_die_offset
	.short	6                       ## DW_FORM_data4
	.long	0                       ## Bucket 0
	.long	2090499946              ## Hash in Bucket 0
	.long	LNames0-Lnames_begin    ## Offset in Bucket 0
LNames0:
	.long	597                     ## main
	.long	1                       ## Num DIEs
	.long	67
	.long	0
	.section	__DWARF,__apple_objc,regular,debug
Lobjc_begin:
	.long	1212240712              ## Header Magic
	.short	1                       ## Header Version
	.short	0                       ## Header Hash Function
	.long	1                       ## Header Bucket Count
	.long	0                       ## Header Hash Count
	.long	12                      ## Header Data Length
	.long	0                       ## HeaderData Die Offset Base
	.long	1                       ## HeaderData Atom Count
	.short	1                       ## DW_ATOM_die_offset
	.short	6                       ## DW_FORM_data4
	.long	-1                      ## Bucket 0
	.section	__DWARF,__apple_namespac,regular,debug
Lnamespac_begin:
	.long	1212240712              ## Header Magic
	.short	1                       ## Header Version
	.short	0                       ## Header Hash Function
	.long	1                       ## Header Bucket Count
	.long	0                       ## Header Hash Count
	.long	12                      ## Header Data Length
	.long	0                       ## HeaderData Die Offset Base
	.long	1                       ## HeaderData Atom Count
	.short	1                       ## DW_ATOM_die_offset
	.short	6                       ## DW_FORM_data4
	.long	-1                      ## Bucket 0
	.section	__DWARF,__apple_types,regular,debug
Ltypes_begin:
	.long	1212240712              ## Header Magic
	.short	1                       ## Header Version
	.short	0                       ## Header Hash Function
	.long	2                       ## Header Bucket Count
	.long	2                       ## Header Hash Count
	.long	20                      ## Header Data Length
	.long	0                       ## HeaderData Die Offset Base
	.long	3                       ## HeaderData Atom Count
	.short	1                       ## DW_ATOM_die_offset
	.short	6                       ## DW_FORM_data4
	.short	3                       ## DW_ATOM_die_tag
	.short	5                       ## DW_FORM_data2
	.short	4                       ## DW_ATOM_type_flags
	.short	11                      ## DW_FORM_data1
	.long	0                       ## Bucket 0
	.long	1                       ## Bucket 1
	.long	193495088               ## Hash in Bucket 0
	.long	2090147939              ## Hash in Bucket 1
	.long	Ltypes0-Ltypes_begin    ## Offset in Bucket 0
	.long	Ltypes1-Ltypes_begin    ## Offset in Bucket 1
Ltypes0:
	.long	602                     ## int
	.long	1                       ## Num DIEs
	.long	123
	.short	36
	.byte	0
	.long	0
Ltypes1:
	.long	616                     ## char
	.long	1                       ## Num DIEs
	.long	145
	.short	36
	.byte	0
	.long	0
	.section	__DWARF,__apple_exttypes,regular,debug
Lexttypes_begin:
	.long	1212240712              ## Header Magic
	.short	1                       ## Header Version
	.short	0                       ## Header Hash Function
	.long	1                       ## Header Bucket Count
	.long	0                       ## Header Hash Count
	.long	12                      ## Header Data Length
	.long	0                       ## HeaderData Die Offset Base
	.long	1                       ## HeaderData Atom Count
	.short	7                       ## DW_ATOM_ext_types
	.short	6                       ## DW_FORM_data4
	.long	-1                      ## Bucket 0

.subsections_via_symbols
	.section	__DWARF,__debug_line,regular,debug
Lsection_line:
Lline_table_start0:
```
