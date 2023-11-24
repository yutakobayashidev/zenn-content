---
title: "Shadcn UIのカレンダーコンポーネントを日本向けにローカライズする"
emoji: "📅"
type: "tech"
topics: ["shadcnui", "react"]
published: true
---

Shadcn UI のカレンダーコンポーネントは、`react-day-picker`というパッケージに依存しています。このカレンダーコンポーネントを日本語化する方法についてです。

## 日本語化する

`date-fns`の日本語ローカライズデータをインポートしてコンポーネントに locale props として渡してあげます。

```tsx
import { ja } from "date-fns/locale";

function Calendar({
  className,
  classNames,
  showOutsideDays = true,
  ...props
}: CalendarProps) {
  return (
    <DayPicker
      locale={ja} // ここでlocaleを渡す
      showOutsideDays={showOutsideDays}
      className={cn("p-3", className)}
      classNames={{
        caption: "flex justify-center pt-1 relative items-center",
        caption_label: "text-sm font-medium",
        cell: "h-9 w-9 text-center text-sm p-0 relative [&:has([aria-selected])]:bg-accent first:[&:has([aria-selected])]:rounded-l-md last:[&:has([aria-selected])]:rounded-r-md focus-within:relative focus-within:z-20",
        day: cn(
          buttonVariants({ variant: "ghost" }),
          "h-9 w-9 p-0 font-normal aria-selected:opacity-100"
        ),
        day_disabled: "text-muted-foreground opacity-50",
        day_hidden: "invisible",
        day_outside: "text-muted-foreground opacity-50",
        day_range_middle:
          "aria-selected:bg-accent aria-selected:text-accent-foreground",
        day_selected:
          "bg-primary text-primary-foreground hover:bg-primary hover:text-primary-foreground focus:bg-primary focus:text-primary-foreground",
        day_today: "bg-accent text-accent-foreground",
        head_cell:
          "text-muted-foreground rounded-md w-9 font-normal text-[0.8rem]",
        head_row: "flex",
        month: "space-y-4",
        months: "flex flex-col sm:flex-row space-y-4 sm:space-x-4 sm:space-y-0",
        nav: "space-x-1 flex items-center",
        nav_button: cn(
          buttonVariants({ variant: "outline" }),
          "h-7 w-7 bg-transparent p-0 opacity-50 hover:opacity-100"
        ),
        nav_button_next: "absolute right-1",
        nav_button_previous: "absolute left-1",
        row: "flex w-full mt-2",
        table: "w-full border-collapse space-y-1",
        ...classNames,
      }}
      components={{
        IconLeft: ({ ...props }) => <ChevronLeft className="h-4 w-4" />,
        IconRight: ({ ...props }) => <ChevronRight className="h-4 w-4" />,
      }}
      {...props}
    />
  );
}
```

## ラベルのフォーマットを日本表記に整える

このまま日本語化するだけだと、年と日付のラベルが「11 月 2023 年」のように、日本人からすると馴染みのない順番の表記になってしまいます。

これを修正するには、date-fns でフォーマットを整えたものを formatters に渡すだけです。

```tsx
import { format } from "date-fns";

function Calendar({
  className,
  classNames,
  showOutsideDays = true,
  ...props
}: CalendarProps) {
  const formatCaption: DateFormatter = (date, options) => {
    const y = format(date, "yyyy");
    const m = format(date, "MM", { locale: options?.locale });
    return `${y}年${m}月`;
  };

  return (
    <DayPicker
      locale={ja}
      formatters={{ formatCaption }} // formattersでラベルの日付形式を修正する
      showOutsideDays={showOutsideDays}
      className={cn("p-3", className)}
      classNames={{
        caption: "flex justify-center pt-1 relative items-center",
        caption_label: "text-sm font-medium",
        cell: "h-9 w-9 text-center text-sm p-0 relative [&:has([aria-selected])]:bg-accent first:[&:has([aria-selected])]:rounded-l-md last:[&:has([aria-selected])]:rounded-r-md focus-within:relative focus-within:z-20",
        day: cn(
          buttonVariants({ variant: "ghost" }),
          "h-9 w-9 p-0 font-normal aria-selected:opacity-100"
        ),
        day_disabled: "text-muted-foreground opacity-50",
        day_hidden: "invisible",
        day_outside: "text-muted-foreground opacity-50",
        day_range_middle:
          "aria-selected:bg-accent aria-selected:text-accent-foreground",
        day_selected:
          "bg-primary text-primary-foreground hover:bg-primary hover:text-primary-foreground focus:bg-primary focus:text-primary-foreground",
        day_today: "bg-accent text-accent-foreground",
        head_cell:
          "text-muted-foreground rounded-md w-9 font-normal text-[0.8rem]",
        head_row: "flex",
        month: "space-y-4",
        months: "flex flex-col sm:flex-row space-y-4 sm:space-x-4 sm:space-y-0",
        nav: "space-x-1 flex items-center",
        nav_button: cn(
          buttonVariants({ variant: "outline" }),
          "h-7 w-7 bg-transparent p-0 opacity-50 hover:opacity-100"
        ),
        nav_button_next: "absolute right-1",
        nav_button_previous: "absolute left-1",
        row: "flex w-full mt-2",
        table: "w-full border-collapse space-y-1",
        ...classNames,
      }}
      components={{
        IconLeft: ({ ...props }) => <ChevronLeft className="h-4 w-4" />,
        IconRight: ({ ...props }) => <ChevronRight className="h-4 w-4" />,
      }}
      {...props}
    />
  );
}
```

## 月曜始まりにする

これはオプションですが、個人的には月曜始まりのほうがしっくり来るので変更しました。weekStartsOn に、数値を日曜始まりから指定できます。

今回は月曜が良いので 1 ですね。

```tsx
function Calendar({
  className,
  classNames,
  showOutsideDays = true,
  ...props
}: CalendarProps) {
  return (
    <DayPicker
      locale={ja}
      weekStartsOn={1} // 数値で曜日を指定
      formatters={{ formatCaption }}
      showOutsideDays={showOutsideDays}
      className={cn("p-3", className)}
      classNames={{
        caption: "flex justify-center pt-1 relative items-center",
        caption_label: "text-sm font-medium",
        cell: "h-9 w-9 text-center text-sm p-0 relative [&:has([aria-selected])]:bg-accent first:[&:has([aria-selected])]:rounded-l-md last:[&:has([aria-selected])]:rounded-r-md focus-within:relative focus-within:z-20",
        day: cn(
          buttonVariants({ variant: "ghost" }),
          "h-9 w-9 p-0 font-normal aria-selected:opacity-100"
        ),
        day_disabled: "text-muted-foreground opacity-50",
        day_hidden: "invisible",
        day_outside: "text-muted-foreground opacity-50",
        day_range_middle:
          "aria-selected:bg-accent aria-selected:text-accent-foreground",
        day_selected:
          "bg-primary text-primary-foreground hover:bg-primary hover:text-primary-foreground focus:bg-primary focus:text-primary-foreground",
        day_today: "bg-accent text-accent-foreground",
        head_cell:
          "text-muted-foreground rounded-md w-9 font-normal text-[0.8rem]",
        head_row: "flex",
        month: "space-y-4",
        months: "flex flex-col sm:flex-row space-y-4 sm:space-x-4 sm:space-y-0",
        nav: "space-x-1 flex items-center",
        nav_button: cn(
          buttonVariants({ variant: "outline" }),
          "h-7 w-7 bg-transparent p-0 opacity-50 hover:opacity-100"
        ),
        nav_button_next: "absolute right-1",
        nav_button_previous: "absolute left-1",
        row: "flex w-full mt-2",
        table: "w-full border-collapse space-y-1",
        ...classNames,
      }}
      components={{
        IconLeft: ({ ...props }) => <ChevronLeft className="h-4 w-4" />,
        IconRight: ({ ...props }) => <ChevronRight className="h-4 w-4" />,
      }}
      {...props}
    />
  );
}
```
