---
{"publish":true,"created":"2026-01-26T06:26:02.000Z","modified":"2026-08-17T03:03:45.349Z"}
---

Project Setup Steps :
npm create vite@latest room-occupancy-optimiser -- --template react-ts

###### Naming Conventions

| Element                  | Convention              | Examples                                                   | Notes                         |
| ------------------------ | ----------------------- | ---------------------------------------------------------- | ----------------------------- |
| **Folders**              | kebab-case or lowercase | `components/`, `utils/`, `user-profile/`, `__tests__/`     | Pick one style, be consistent |
| **Component Files**      | PascalCase              | `RoomOptimizer.tsx`, `UserProfile.tsx`, `NavBar.tsx`       | Must match component name     |
| **Utils/Helpers Files**  | camelCase               | `roomOptimization.ts`, `formatCurrency.ts`, `apiHelper.ts` | -                             |
| **Test Files**           | Source name + `.test`   | `RoomOptimizer.test.tsx`, `roomOptimization.test.ts`       | Match the file being tested   |
| **Type/Interface Files** | camelCase               | `types.ts`, `roomTypes.ts`, `userTypes.ts`                 | File names are camelCase      |
| **Component Names**      | PascalCase              | `RoomOptimizer`, `Button`, `UserCard`                      | Match file name exactly       |
| **Functions/Methods**    | camelCase               | `optimizeRooms()`, `calculateRevenue()`, `handleClick()`   | -                             |
| **Variables**            | camelCase               | `premiumRooms`, `totalRevenue`, `guestList`                | -                             |
| **Constants**            | UPPER\_SNAKE\_CASE        | `MAX_ROOMS`, `DEFAULT_PRICE`, `API_URL`                    | Only for true constants       |
| **Interfaces**           | PascalCase              | `RoomResult`, `GuestPrice`, `UserProfile`                  | Always PascalCase             |
| **Types**                | PascalCase              | `Status`, `RoomCategory`, `PriceRange`                     | Always PascalCase             |
| **Enums**                | PascalCase              | `RoomType`, `BookingStatus`                                | Enum name and values          |
| **Boolean Variables**    | is/has                  |                                                            |                               |
