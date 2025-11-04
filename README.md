/**
 * Carousel - Componente de Carrossel Vanilla JavaScript
 * 
 * Funcionalidades:
 * - Navegação com botões (próximo/anterior)
 * - Indicadores clicáveis
 * - Autoplay configurável
 * - Controle por teclado (setas)
 * - Transições CSS customizáveis (fade, slide, zoom)
 * - Responsivo
 */

class Carousel {
  constructor(options = {}) {
    // Configurações padrão
    this.container = options.container || document.querySelector('.carousel-container');
    this.autoPlay = options.autoPlay !== false;
    this.autoPlayInterval = options.autoPlayInterval || 5000;
    this.transitionDuration = options.transitionDuration || 500;
    this.transitionType = options.transitionType || 'slide';
    this.showIndicators = options.showIndicators !== false;
    this.showControls = options.showControls !== false;
    this.onItemChange = options.onItemChange || null;

    if (!this.container) {
      console.error('Carousel: container não encontrado');
      return;
    }

    // Estado
    this.currentIndex = 0;
    this.isTransitioning = false;
    this.autoPlayTimer = null;

    // Elementos
    this.wrapper = this.container.querySelector('.carousel-wrapper');
    this.slides = this.container.querySelectorAll('.carousel-slide');
    this.prevButton = this.container.querySelector('.carousel-button-prev');
    this.nextButton = this.container.querySelector('.carousel-button-next');
    this.indicatorsContainer = this.container.querySelector('.carousel-indicators');

    if (this.slides.length === 0) {
      console.warn('Carousel: nenhum slide encontrado');
      return;
    }

    this.init();
  }

  init() {
    // Aplicar classe de transição
    if (this.wrapper) {
      this.wrapper.classList.add(this.transitionType);
      this.wrapper.style.transitionDuration = `${this.transitionDuration}ms`;
    }

    // Event listeners - Botões
    if (this.prevButton) {
      this.prevButton.addEventListener('click', () => this.goToPrev());
    }
    if (this.nextButton) {
      this.nextButton.addEventListener('click', () => this.goToNext());
    }

    // Event listeners - Indicadores
    if (this.indicatorsContainer) {
      const indicators = this.indicatorsContainer.querySelectorAll('.carousel-indicator');
      indicators.forEach((indicator, index) => {
        indicator.addEventListener('click', () => this.goToSlide(index));
      });
    }

    // Event listeners - Teclado
    document.addEventListener('keydown', (e) => this.handleKeydown(e));

    // Mostrar primeiro slide
    this.showSlide(0);

    // Iniciar autoplay
    if (this.autoPlay) {
      this.startAutoPlay();
    }

    // Pausar autoplay ao passar o mouse
    this.container.addEventListener('mouseenter', () => this.pauseAutoPlay());
    this.container.addEventListener('mouseleave', () => {
      if (this.autoPlay) this.startAutoPlay();
    });
  }

  showSlide(index) {
    // Remover classe active de todos os slides
    this.slides.forEach(slide => slide.classList.remove('active'));

    // Adicionar classe active ao slide atual
    if (this.slides[index]) {
      this.slides[index].classList.add('active');
    }

    // Atualizar indicadores
    if (this.indicatorsContainer) {
      const indicators = this.indicatorsContainer.querySelectorAll('.carousel-indicator');
      indicators.forEach((indicator, i) => {
        indicator.classList.toggle('active', i === index);
      });
    }

    // Callback
    if (this.onItemChange) {
      this.onItemChange(index);
    }
  }

  goToNext() {
    if (!this.isTransitioning) {
      this.isTransitioning = true;
      this.currentIndex = (this.currentIndex + 1) % this.slides.length;
      this.showSlide(this.currentIndex);
      this.resetTransitionTimeout();
    }
  }

  goToPrev() {
    if (!this.isTransitioning) {
      this.isTransitioning = true;
      this.currentIndex = (this.currentIndex - 1 + this.slides.length) % this.slides.length;
      this.showSlide(this.currentIndex);
      this.resetTransitionTimeout();
    }
  }

  goToSlide(index) {
    if (!this.isTransitioning && index !== this.currentIndex) {
      this.isTransitioning = true;
      this.currentIndex = index;
      this.showSlide(this.currentIndex);
      this.resetTransitionTimeout();
    }
  }

  resetTransitionTimeout() {
    setTimeout(() => {
      this.isTransitioning = false;
    }, this.transitionDuration);
  }

  handleKeydown(e) {
    if (e.key === 'ArrowLeft') {
      this.goToPrev();
    } else if (e.key === 'ArrowRight') {
      this.goToNext();
    }
  }

  startAutoPlay() {
    if (this.autoPlayTimer) return;

    this.autoPlayTimer = setInterval(() => {
      this.currentIndex = (this.currentIndex + 1) % this.slides.length;
      this.showSlide(this.currentIndex);
    }, this.autoPlayInterval);
  }

  pauseAutoPlay() {
    if (this.autoPlayTimer) {
      clearInterval(this.autoPlayTimer);
      this.autoPlayTimer = null;
    }
  }

  destroy() {
    this.pauseAutoPlay();
    if (this.prevButton) {
      this.prevButton.removeEventListener('click', () => this.goToPrev());
    }
    if (this.nextButton) {
      this.nextButton.removeEventListener('click', () => this.goToNext());
    }
    document.removeEventListener('keydown', (e) => this.handleKeydown(e));
  }
}

// Inicializar carrossel quando o DOM estiver pronto
document.addEventListener('DOMContentLoaded', () => {
  const carousels = document.querySelectorAll('.carousel-container');
  carousels.forEach(carousel => {
    const transitionType = carousel.dataset.transition || 'slide';
    const autoPlayInterval = parseInt(carousel.dataset.autoPlayInterval) || 5000;

    new Carousel({
      container: carousel,
      transitionType: transitionType,
      autoPlayInterval: autoPlayInterval,
      onItemChange: (index) => {
        // Callback para atualizar informações do slide
        const slideCounter = carousel.parentElement?.querySelector('.slide-counter');
        if (slideCounter) {
          const totalSlides = carousel.querySelectorAll('.carousel-slide').length;
          slideCounter.textContent = `${index + 1} de ${totalSlides}`;
        }
      }
    });
  });
});
