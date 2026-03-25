# Dev_course 프로젝트 1 진행중
# 장바구니의 전역 상태로 만드는 과정에 대한 설명

프론트에서 협업이 익숙하지 않았다. 상품을 보여주는 ProductList.tsx나 Cart.tsx, Order.... 등 이런식으로 구분이 되어있었다. 처음에는 내가 상품 UI를 알지도 못하면 Cart부분을 구현할 수 없을 줄 알았다. 하지만 많이 잘못된 생각이었다.

<br>

프론트에서 전역 상태 관리라는 것이 잘 이해가 가지 않았다. 백엔드는 전역 변수라는 개념으로 공통적으로 사용할 수 있다는 것을 알고 있었지만 프론트에서 어떻게 가능한지 감이 잘 잡히지 않았다.

<br>

일단 감을 잡기 위해 어떻게 돌아가는지 정리할 것이다.

# 전역 상태 관리하는 과정

일단 우리 팀 프로젝트의 계층 구조는 아래 이미지와 같다. 

![전역1](/Front/front_images/전역1.png)

저기서 이해가 안되는 부분이 Context 부분이었다. 하지만 저기서 cart 안에 들어갈때 사용되는 함수나 변수, 로직들을 묶어놓는다고 생각하면 편하다.

<details>
<summary>코드 살펴보기</summary>

일부로 생략 없이 적었다. 추후에 내가 잘 이해하기 위해서

> context/CartContext.tsx

    "use client";
    import React, { createContext, useContext, useState, useMemo } from "react";

    interface CartContextType {
        cart: { [key: number]: number };
        updateQuantity: (id: number, delta: number) => Promise<void>;
        totalAmount: number;
        resetCart: () => void;
    }

    const CartContext = createContext<CartContextType | undefined>(undefined);

    export function CartProvider({ children }: { children: React.ReactNode }) {
        
        const [products, setProducts] = useState<Product[]>([]);// 백엔드에서 받을 전체 목록

        const [cart, setCart] = useState<{ [key: number]: number }>({});

        // 상품 목록 데이터 (ProductList 담당자와 규격을 맞춘 데이터) 
        //임시 데이터임.
        const products = [
            { id: 1, name: "에티오피아 예가체프", price: 15000 },
            { id: 2, name: "콜롬비아 수프리모", price: 13000 },
            { id: 3, name: "브라질 산토스", price: 12000 },
            { id: 4, name: "과테말라 안티구아", price: 14000 },
        ];

        // 페이지 로드 시 딱 한 번 실행
        //이것을 추가하면 임시데이터를 제거해도된다. 
        useEffect(() => {
            fetch("http://localhost:8080/api/v1/products")
            .then(res => res.json())
            .then(data => setProducts(data)); // 여기서 정보를 채워넣음!
        }, []);


        const updateQuantity = async (id: number, delta: number) => {
            const currentQty = cart[id] || 0;
            const targetQty = currentQty + delta;

            //입력 받은 값이 -1 즉, 줄이는 값이라면 그냥 api 호출없이 값을 줄인다.
            if (delta < 0) {
                setCart((prev) => ({ ...prev, [id]: Math.max(0, targetQty) }));
                return;
            }

            try {
                const response = await fetch(`도메인 주소/api/v1/product/${id}/stock?requestQuantity=${targetQty}`);
                if (!response.ok) {
                    const result = await response.json();
                    alert(result.error.message || "재고가 부족합니다.");
                    return;
                }
                setCart((prev) => ({ ...prev, [id]: targetQty }));
            } catch (e) {
                alert("서버와 통신 중 오류가 발생했습니다.");
            }
        };

        const totalAmount = useMemo(() => {
            return products.reduce((acc, p) => acc + (p.price * (cart[p.id] || 0)), 0);
        }, [cart]);

        const resetCart = () => setCart({});

        return (
            <CartContext.Provider value={{ cart, updateQuantity, totalAmount, resetCart }}>
                {children}
            </CartContext.Provider>
        );
    }

    export const useCart = () => {
        const context = useContext(CartContext);
        if (!context) throw new Error("useCart must be used within CartProvider");
        return context;
    };


우리가 핵심적으로 봐야 할 것은 CartProvider이다. 
<br>
여기 내부에는 여러가지 함수가 존재하는데 정리하자면 cart, updateQuantity, totalAmount, resetCart이다.


각 기능에 대해 설명하자면

> cart

cart는 고객이 선택한 아이템을 담는 바구니라고 생각하면 된다.

    {
        "1": 2,  // 에티오피아 예가체프 2개
        "3": 1,  // 브라질 산토스 1개
        "4": 5   // 과테말라 안티구아 5개
    }
이런 형태로 들어간다고 보면된다.

> updateQuantity

상품의 아이디와 수량이 입력되면 cart 변수에 값을 넣는 함수이다. 거기에다 백엔드 재고 조회 api가 연결되어서 재고가 없을 경우 재고가 없다는 창을 띄우는 로직이라고 보면된다. 

> totalAmount

카트에 담긴 금액이 얼마인지 파악하는 로직이다.

> resetCart

만약 주문 오더가 들어갈 경우 장바구니에 있는 내용은 전부 사라져야 하기 때문에 그 cart를 비우는 로직이다.

</details>

<br>

위에서 설명한 코드를(각 기능들 cart,resetcart 등...) 다른 팀원이 사용하게끔 만드는 것이 **전역 상태**로 바꾸는 것(?)이다.

그러기 위해서는 계층 구조 이미지의 app/layout.tsx에서 작업을 해줘야 한다.

> app/layout.tsx

    import { CartProvider } from "@/context/CartContext";

    export default function RootLayout({ children }: { children: React.ReactNode }) {
    return (
        <html lang="ko">
        <body className="bg-amber-50/50 min-h-screen">
            <CartProvider> {/* 여기에 배치 */}
            <header className="...">...</header>
            {children}
            </CartProvider>
        </body>
        </html>
    );
    }

layout은 전체 프론트의 구조를 설정하는 파일이라고 보면 된다. 여기서 header와 children을 CartProvider로 감싸면 준비가 끝난다.

# 다른 팀원이 사용하는 방법

라이브러리화 시켰다고 생각하면 된다. 예를 들어 productList를 담당하는 팀원이 화면에 이미지를 뿌렸는데 고객이 클릭하면 장바구니로 들어가게끔 구현해야 한다. 어떻게 가능하게 할까? CartContext.tsx의 마지막 부분을 보면 UseCart라는 부분이 있다. 이것을 사용해서 내가 정의한 기능들을 사용할 수 있게 된다.

<details>

<summary>코드로 살펴보자</summary>

    "use client";
    // 1. 만든 훅을 임포트합니다.
    import { useCart } from "@/context/CartContext"; 

    export default function ProductList() {
    // 2.  필요한 함수만 쏙 빼온다.
    const { updateQuantity } = useCart();

    const products = [ /* 상품 데이터들... */ ];


    //상품 데이터를 화면에 보여주는 코드
    return (
        <div className="grid grid-cols-2 gap-4">
        {products.map((product) => (
            <div key={product.id} className="border p-4">
            <h4>{product.name}</h4>
            <button 
                onClick={() => updateQuantity(product.id, 1)} // 3. 여기서 함수를 호출해준다.
                className="bg-stone-800 text-white p-2"
            >
                장바구니 담기
            </button>
            </div>
        ))}
        </div>
    );
    }

CartContext에서 만들었던 updateQuantity를 UseCart를 이용해서 위처럼 사용이 가능하다. 이제 고객이 이미지를 클릭할 경우 상품의 아이디와 숫자값이 넘어가면서 cart내부의 값이 변경될 것이다. 

<br>
<br>

그리고 장바구니를 담당한 사람은 cart를 보여주는 코드를 짜면된다. 왜냐하면 cart 내부의 값을 전역적으로 공유하고 있기 때문에 가능한 일이다.

## components/Cart.tsx

    "use client";

    import { useCart } from "@/context/CartContext";
    import { Product } from "@/types/product"; // 타입은 별도 파일로 관리.

    export default function Cart() {

    // 1. CartContext에서 필요한 데이터와 함수만 쏙 골라서 가져옵니다.
    const { cart, updateQuantity, totalAmount } = useCart();

    // 실제 화면에 그릴 상품 리스트 (Context 내부나 상위에서 관리되는 전체 상품 목록)
    const products: Product[] = [
        { id: 1, name: "에티오피아 예가체프", price: 15000 },
        { id: 2, name: "콜롬비아 수프리모", price: 13000 },
        { id: 3, name: "브라질 산토스", price: 12000 },
        { id: 4, name: "과테말라 안티구아", price: 14000 },
    ];

    return (
        <div className="w-full">
        <h3 className="text-lg font-bold mb-4 text-stone-800">장바구니</h3>
        
        {/* 1. 장바구니 리스트 영역 */}
        <div className="bg-white rounded-lg border border-stone-200 divide-y divide-stone-100">
            {totalAmount === 0 ? (
            <div className="py-12 text-center">
                <p className="text-sm text-stone-400">장바구니가 비어 있습니다.</p>
            </div>
            ) : (
            <div className="p-4 space-y-4">
                {products.map((p) => {
                const quantity = cart[p.id] || 0;
                if (quantity <= 0) return null;

                return (
                    <div key={p.id} className="flex flex-col gap-2 pb-4 last:pb-0">
                    <div className="flex justify-between items-start">
                        <div className="flex flex-col">
                        <span className="text-sm font-medium text-stone-700">{p.name}</span>
                        <span className="text-xs text-stone-400">{p.price.toLocaleString()}원</span>
                        </div>
                        <span className="text-sm font-bold text-stone-900">
                        {(p.price * quantity).toLocaleString()}원
                        </span>
                    </div>

                    {/* 수량 조절 컨트롤러 */}
                    <div className="flex justify-end">
                        <div className="flex items-center bg-stone-50 rounded-full border border-stone-200 p-1">
                        <button 
                            onClick={() => updateQuantity(p.id, -1)}
                            className="w-7 h-7 flex items-center justify-center rounded-full bg-white border border-stone-200 text-stone-600 hover:bg-stone-100 transition-colors"
                        >
                            －
                        </button>
                        <span className="text-sm font-bold text-stone-700 w-8 text-center">
                            {quantity}
                        </span>
                        <button 
                            onClick={() => updateQuantity(p.id, 1)}
                            className="w-7 h-7 flex items-center justify-center rounded-full bg-white border border-stone-200 text-stone-600 hover:bg-stone-100 transition-colors"
                        >
                            ＋
                        </button>
                        </div>
                    </div>
                    </div>
                );
                })}
            </div>
            )}
        </div>

        {/* 합계 요약 (OrderForm으로 넘어가기 전 정보 제공) */}
        {totalAmount > 0 && (
            <div className="mt-4 px-2 flex justify-between items-center">
            <span className="text-sm text-stone-500">선택 상품 합계</span>
            <span className="text-lg font-extrabold text-stone-900">
                {totalAmount.toLocaleString()}원
            </span>
            </div>
        )}
        </div>
    );
    }


위의 코드도 잘 살펴보면 useCart를 이용해서 CartContext에 선언한 함수를 사용하는 것을 볼 수 있다. <br>
저기서 보여지는 값은 전역적으로 공유되는 cart라는 것이다 그래서 productList에서 클릭한 값이 바로 장바구니로 들어가는 것처럼 보이게 되는 것이다.

</details>

