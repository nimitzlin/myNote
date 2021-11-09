遇到一台不支持avx3的机器的记录
一段测试代码

#include <immintrin.h>

int main(int argc, char **argv) {
    __m128 a;
    __m128 b;
    __m128 c;

    c = _mm_add_ps(a, b);
    unsigned int num_buckets = 1;
    unsigned int new_buckets = num_buckets ? new_buckets :16;

    return 0;
}

clang -S main.cpp -o a.s -mavx2 -mfma
编译汇编如下


.Ltmp2:
    .cfi_def_cfa_register %rbp
    movl    $0, -36(%rbp)
    movl    %edi, -40(%rbp)
    movq    %rsi, -48(%rbp)
    vmovaps -64(%rbp), %xmm0
    vmovaps -80(%rbp), %xmm1
    vmovaps %xmm0, -16(%rbp)
    vmovaps %xmm1, -32(%rbp)
    vmovaps -16(%rbp), %xmm0
    vaddps  -32(%rbp), %xmm0, %xmm0
    vmovaps %xmm0, -96(%rbp)
    movl    $1, -100(%rbp)
    cmpl    $0, -100(%rbp)
    je  .LBB0_2

    movl    -104(%rbp), %eax
    movl    %eax, -108(%rbp)        # 4-byte Spill
    jmp .LBB0_3

clang -S main.cpp -o b.s -march=native
编译汇编如下

.Ltmp2:
    .cfi_def_cfa_register %rbp
    movl    $0, -36(%rbp)
    movl    %edi, -40(%rbp)
    movq    %rsi, -48(%rbp)
    movaps  -64(%rbp), %xmm0
    movaps  -80(%rbp), %xmm1
    movaps  %xmm0, -16(%rbp)
    movaps  %xmm1, -32(%rbp)
    movaps  -16(%rbp), %xmm0
    addps   -32(%rbp), %xmm0
    movaps  %xmm0, -96(%rbp)
    movl    $1, -100(%rbp)
    cmpl    $0, -100(%rbp)
    je  .LBB0_2

    movl    -104(%rbp), %eax
    movl    %eax, -108(%rbp)        # 4-byte Spill
    jmp .LBB0_3



jhlin@g20-common:~/vmovsstest$ clang main.cpp -mavx2 -mfma
jhlin@g20-common:~/vmovsstest$ ./a.out
Illegal instruction (core dumped)
jhlin@g20-common:~/vmovsstest$ clang main.cpp -march=native
jhlin@g20-common:~/vmovsstest$ ./a.out
无错误
