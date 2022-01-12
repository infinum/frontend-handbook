This chapters covers quirks of working with and testing ResizeObserver in Angular.

## Resources

Repository with a [working example](https://github.com/infinum/JS-ResizeObserver-Angular).

## Issue

You have a piece of DOM, which size you need to keep track of. ResizeObserver makes such things trivial.

```ts
// ❌ incorrect
@Component({
  selector: 'app-example',
  template: `
    <div #target></div>
    <output *ngIf="height$ | async as height">{{ height }}</output>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
class ExampleComponent implements AfterViewInit, OnDestroy {
  @ViewChild('target', { static: true, read: ElementRef })
  public readonly target!: ElementRef<HTMLElement>;
  public readonly resizeObserver?: ResizeObserver;
  private readonly _height$ = new BehaviorSubject(0);
  public readonly height$ = this._height.asObservable();

  public ngAfterViewInit(): void {
    this.resizeObserver = new ResizeObserver((entries) => {
      if (entries.length) {
        this._height$.next(entries[0].contentRect.height);
      }
    });
    this.resizeObserver.observe(this.target.nativeElement);
  }

  public ngOnDestroy(): void {
    this.resizeObserver?.disconnect();
  }
}
```

Unfortunately above won't work correctly within Angular. Even though some [resources](https://github.com/angular/zone.js/issues/1011) may say otherwise, the callback provided to ResizeObserver doesn't run within NgZone and thus after finishing it doesn't run change detection cycle.

To check that yourself, you may use static method isInAngularZone of NgZone like so:

```ts
new ResizeObserver((entries) => {
  console.log(NgZone.isInAngularZone()); // logs out false when triggered
});
```

Therefore that makes it necessary to reenter the zone manually using NgZone, like so:

## Solution

```ts
// ✅ correct
@Component({
  selector: 'app-example',
  template: `
    <div #target></div>
    <output *ngIf="height$ | async as height">{{ height }}</output>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
class ExampleComponent implements AfterViewInit, OnDestroy {
  @ViewChild('target', { static: true, read: ElementRef })
  public readonly target!: ElementRef<HTMLElement>;
  public readonly resizeObserver?: ResizeObserver;
  private readonly _height$ = new BehaviorSubject(0);
  public readonly height$ = this._height.asObservable();

  // inject zone
  constructor(private readonly ngZone: NgZone) {}

  public ngAfterViewInit(): void {
    // reenter zone in callback
    this.resizeObserver = new ResizeObserver((entries) =>
      this.ngZone.run(() => {
        if (entries.length) {
          this._height$.next(entries[0].contentRect.height);
        }
      })
    );
    this.resizeObserver.observe(this.target.nativeElement);
  }

  public ngOnDestroy(): void {
    this.resizeObserver?.disconnect();
  }
}
```

With that we made sure that \_height$ latest value is actually rendered.

## Test issue

Testing the above has its own quirks as well.

Suppose we attempt to test the ResizeObserver with following cases:

```ts
// ❌ incorrect
describe('AppComponent', () => {
  let fixture: ComponentFixture<AppComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [RouterTestingModule],
      declarations: [AppComponent],
    }).compileComponents();
    fixture = TestBed.createComponent(AppComponent);
    fixture.detectChanges();
  });

  it('should mirror rectangle height into output', async () => {
    const resizableDebugEl = fixture.debugElement.query(By.css('div'));
    const heightOutputDebugEl = fixture.debugElement.query(By.css('output'));

    let newHeight = '250';
    resizableDebugEl.nativeElement.style.height = `${newHeight}px`;

    await fixture.whenStable();
    fixture.detectChanges();

    expect(heightOutputDebugEl.nativeElement.innerText).toBe(newHeight);

    newHeight = '400';
    resizableDebugEl.nativeElement.style.height = `${newHeight}px`;

    await fixture.whenStable();
    fixture.detectChanges();

    expect(heightOutputDebugEl.nativeElement.innerText).toBe(newHeight);
  });
});
```

Unfortunately above won't as well as when we run the tests, we will see it complaining that expected `heightOutputDebugEl` innerText doesn't match `newHeight`. Why is that?

Clearly the browser doesn't execute the ResizeObserver callback the instant one of the elements gets resized or to make it clearer, it doesn't get called the moment anything which could cause a resize happens. Instead, it runs as described in [spec](https://www.w3.org/TR/resize-observer/#html-event-loop), TL;DR after layout gets updated, but before repaint. Closest the end of that would be using `requestAnimationFrame` with `setTimeout`, where `requestAnimationFrame` runs its callback just before the repaint happens, we can defer it to next macrotask with `setTimeout` (which is bound to happen after repaint). We could use artificial delay as well, but that wouldn't guarantee the execution after the next repaint.

## Test solution

```ts
// ✅ correct
function afterRepaint(): Promise<void> {
  return new Promise((resolve) => {
    requestAnimationFrame(() => setTimeout(() => resolve()));
  });
}

describe('AppComponent', () => {
  let fixture: ComponentFixture<AppComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [RouterTestingModule],
      declarations: [AppComponent],
    }).compileComponents();
    fixture = TestBed.createComponent(AppComponent);
    fixture.detectChanges();
  });

  it('should mirror rectangle height into output', async () => {
    const resizableDebugEl = fixture.debugElement.query(By.css('div'));
    const heightOutputDebugEl = fixture.debugElement.query(By.css('output'));

    let newHeight = '250';
    resizableDebugEl.nativeElement.style.height = `${newHeight}px`;

    await afterRepaint();
    await fixture.whenStable();
    fixture.detectChanges();

    expect(heightOutputDebugEl.nativeElement.innerText).toBe(newHeight);

    newHeight = '400';
    resizableDebugEl.nativeElement.style.height = `${newHeight}px`;

    await afterRepaint();
    await fixture.whenStable();
    fixture.detectChanges();

    expect(heightOutputDebugEl.nativeElement.innerText).toBe(newHeight);
  });
});
```

By awaiting `afterRepaint` we are guaranteeing that the test case is synchronized properly.
